---
layout: post
title: Wrastle with pgpool and postgres
---
First thing first. To setup postgres instances in primary standby mode, you need at least two instances of postgres to play with. I am going to work with postgres version 9.3 and my host operating system in ubuntu.
Creating postgres cluster instances

Assuming you have postgres installed. Here is a simple utility that can give you multiple instances of postgres in a minute. Here is what I ran.

    pg_createcluster 9.3 pgpool_cluster_standby --start-conf manual

    pg_createcluster 9.3 pgpool_cluster --start-conf manual

not this command shall require either root( or sudo privilege). Alternatively you can login into postgres user to perform these actions.To see detailed controls that this utility provide, try pg_createcluster --help

The above command gives me two postgres instances on port 5433 and 5434.

To see the details about the postgres instances running on your machine, you can use below command

    pg_lscluster

Here is the output I got

    postgres@ubuntu:/etc/postgresql/9.3/pgpool_cluster$ pg_lsclusters
    Ver Cluster Port Status Owner Data directory Log file
    9.3 main 5432 down postgres /var/lib/postgresql/9.3/main /var/log/postgresql/postgresql-9.3-main.log
    9.3 pgpool_cluster 5433 down postgres /var/lib/postgresql/9.3/pgpool_cluster /var/log/postgresql/postgresql-9.3-pgpool_cluster.log
    9.3 pgpool_cluster_standby 5434 down postgres /var/lib/postgresql/9.3/pgpool_cluster_standby /var/log/postgresql/postgresql-9.3-pgpool_cluster_standby.log

Note that configuration files for postgres are by default on this path on ubuntu /etc/postgres/<version>/. In our case version is 9.3
Configuring postgres instances to run in streaming replication mode

This is going to be a series of steps that I needed to perform. First two actions are required to be performed on both the instances of postgres.
AllowingConnectionsToPostgres
ChangePassword
ConfiguringFiles
CreatingBaseBackupAndRestoringItToStandby
SettingUpFailoverScript

Allowing Connections To Postgres

You need to allow connections to your newly created instances. To understand what I mean by this, here is what I propose you to do. Try these out:

Login into postgres user from terminal

    sudo su - postgres

start pgpool_cluster instance

    pg_ctlcluster 9.3 pgpool_cluster start

_now try

    psql

You shall see error like this:

    psql: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"?

Now, once you have seen this, stopping the postgres instance that you just started. Use pg_ctlcluster utility for the same.

To get past this issue, make sure that pg_hba.conf file on this ConfigFilesPath have connection mode set as trust. (Note: this only for setting up postgres in dev environment. For production environment, this file needs to be very much fine tuned)

Note: I am making change for replication privileges too here.

Here is what you need to edit. (note: I have uncommented last two lines)

    local all postgres trust
    #TYPE DATABASE USER ADDRESS METHOD
    #"local" is for Unix domain socket connections only
    local all all trust
    #IPv4 local connections:
    host all all 127.0.0.1/32 trust
    #IPv6 local connections:
    host all all ::1/128 md5
    #Allow replication connections from localhost, by a user with the
    #replication privilege.
    local replication postgres trust
    host replication postgres 127.0.0.1/32 trust
    #host replication postgres ::1/128 md5

You shall still see the error1 mentioned above if you still try to perform above steps. Hold on still. We need to change the password too.

Changing the password

To gain access to newly created instances you shall need to change the password for user postgres for newly created instances.Perform these steps. Note, if you are already logged in the terminal with postgres user, then you can skip through step 1 below.

    sudo su - postgres
    psql -U postgres -h localhost -p 5433
    alter user postgres with password '<newpassword>';

Configuring files

I am going to make pgpool_cluster (port 5433) as my primary and pgpool_cluster_standby (port 5434) as my hot standby server. With this in mind here are the changes I am going to do on this file /etc/postgresql/9.3/pgpool_cluster/postgresql.conf at path ConfigFilesPath

bq. wal_level = hot_standby
bq. checkpoint_segments = 10
bq. checkpoint_completion_target = 0.9
bq. max_wal_senders = 10
bq. wal_keep_segments = 100
bq. hot_standby = on

Similarly file /etc/postgresql/9.3/pgpool_cluster_standby/postgresql.conf needs to be changed as above.

Once this is done, then on data directory we need to create a file with name recovery.conf with a content similar to this.

    #If "recovery.conf" is present in the PostgreSQL data directory, it is
    #read on postmaster startup. After successful recovery, it is renamed
    #to "recovery.done" to ensure that we do not accidentally re-enter
    #archive recovery or standby mode.
    #
    standby_mode = 'on'
    primary_conninfo = 'host=<master host ip> port=<master port> user=<recovery user> password=<password for recovery user>'
    trigger_file = '<data directory>/trigger_file'

(NOTE IT: VERY IMPORTANT) On master, name this file as recovery.done, while on standby name this file as recovery.conf. Also note, for recovery.done, conninfo shall be for standby; while for standby conninfo shall be of primary.

For example, this is the content of my recovery.done file that I am keeping in /var/lib/postgresql/9.3/pgpool_cluster path.

    standby_mode = 'on'
    primary_conninfo = 'host=localhost port=5434 user=postgres password=zantaz'
    trigger_file = '/var/lib/postgresql/9.3/pgpool_cluster/postgresql_pgpool_cluster.trigger'

And this recovery.conf that I am keeping on this path {{}}

    standby_mode = 'on'
    primary_conninfo = 'host=localhost port=5433 user=postgres password=zantaz'
    trigger_file = '/var/lib/postgresql/9.3/pgpool_cluster/postgresql_pgpool_cluster_standby.trigger'

Creating BaseBackup And Restoring It ToStandby

This is what is desired.

    pg_ctlcluster 9.3 pgpool_cluster start;

    /usr/lib/postgresql/9.3/bin/pg_basebackup -h localhost -p 5433 -U postgres -D ~/primary.backup -X stream

    cd /var/lib/postgresql/9.3/pgpool_cluster_standby

    cp ./recovery.conf ~/

    rm -rf *

    cp -R ~/primary.backup/* .

    cp ~/recovery.conf .

    pg_ctlcluster 9.3 pgpool_cluster_standby start

Now your postgres servers are running. And one is primary and other is standby.

This is one good link that tells you how to restore standby from the basebackup of primary., follow this link here

Setting Up Failover Script

I haven't been able to do it. If you have any idea, please feel free to add that information here. (Some hint: some failover scripts have to be setup as part of this; Also I am not sure how to do the recovery in case of failover)
Postgres Streaming Replication

Now few words on High Availability,Load Balancing, and Replication of
postgres.

There are two types of replication for postgres. Warm Standby (or log
shipping) and Stream Replication (hot standby)

Read about them here. http://www.postgresql.org/docs/9.3/static/warm-standby.html

Some notes for setting Postgres for Streaming Replication:
wal_keep_segments: on master are kept to some high value to ensure
that old WAL segments are not recycled soon

archive_timeout: not required

ALTERNATIVELY : If you set up a WAL archive that's accessible from the
standby, wal_keep_segments is not required as the standby can always
use the archive to catch up.

To use streaming replication,

    set up a file-based log-shipping standby server

    The step that turns a file-based log-shipping standby into streaming
    replication standby is setting primary_conninfo setting in the
    recovery.conf file to point to the primary server

    On systems that support the keepalive socket option, setting
    tcp_keepalives_idle, tcp_keepalives_interval and tcp_keepalives_count
    helps the primary promptly notice a broken connection.

    Set the maximum number of concurrent connections from the standby
    servers (see max_wal_senders for details).

    When the standby is started and primary_conninfo is set correctly, the
    standby will connect to the primary after replaying all WAL files
    available in the archive. If the connection is established
    successfully, you will see a walreceiver process in the standby, and a
    corresponding walsender process in the primary.

Note: recovery.conf, can be on standby only. For primary, this file should be
named as recover.done

Once, I collected the whole knowldege of above. And did the configurations
right,

Some directories to look for, while streaming replication is being done are:
<postgres/data/dir>/pg_xlog/
See also

Restoring basebackup on standby
A note regarding recovery.conf
