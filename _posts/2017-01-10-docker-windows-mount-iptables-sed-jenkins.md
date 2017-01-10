---
layout: post
title: Some notes on docker, windows mount in linux, iptables, sed and jenkins
---
docker inspect
docker exec -ti 
doceker rm -v
docker rmi


docker inspect   --format='{{.Config.Env}}  {{.Config.Entrypoint}}' e8f63f961ed8


mvn dependency:build-classpath
mvn compile
mvn clean
mvn install
mvn package
mvn build


Command to run archive on local system that didn't work.
java -Xmx256m -Dcaf.appname=aspen/archive/v1 -Dconfig.rest.host=http://16.103.34.142:80/ -cp "$CLASSPATH":target/archive-1.0.jar com.hpe.aspen.archive.StorageApplication  server aspen-archive.yaml

# Command to mount NextGen windows share on centos
sudo  mount -t cifs //16.184.22.199/BigData/ /mnt/aspen_share/ -o username=bigdata1,password=bigdata1

To drop a schema in vertica

drop schema aspen.account_49874150889620480 cascade

git log -p <fileName>

GIT WORKFLOW

I created a branch ASP-1413.
Then made my changes. Had multile commits there.
Then I went to sprint-26 branch
Ran git merge ASP-1413 // this merges my changes with the branch sprint-26.
then I trid to sqash my changes by git rebase -i origin/sprint-26 // while I am still on sprint-26 branch.
but then I hit the problem Cannot 'squash' without a previous commit message. 

at this point when I saw the git branch, it was in some rebasing state.

so I had to do git rebase --abort

by this I could return back to sprint-26 branch. Now I have to again work to squash changes.


THis is the link that helped my coming out of this rebase thing situation: http://stackoverflow.com/questions/2563632/how-can-i-merge-two-commits-into-one

########################################################

To create user in vertica: dbadmin-> create user akhilesh identified by 'User1234';


########################################################

Sometime while using volumes with docker, you might see that docker container is not able to see the mounted volume inside it.
ls -l  shall give you permissions like 

1000 1000

For that case, use privileged=true option with docker run. Like this: [developer@localhost:1 ~/repo/aspen/aspen/aspen-containers/admin-ui-container/target]$  sudo docker run --privileged=true -d -p 63372:8080 -v /tmp/:/usr/local/tomcat/webapps/config aspen/admin-ui-container:latest

########################################################

#####################################################################
To start tomcat in debug mode
#####################################################################

Add below variable, to CATALINA_HOME/bin/setenv.sh

	export JPDA_OPTS="-agentlib:jdwp=transport=dt_socket, address=1043, server=y, suspend=n"


#################################################################
To kill vertica sessions, you can use this command

select E'SELECT CLOSE_SESSIOn(\'' || session_id || E'\');' from sessions;

This shall return you a list output. Select session that you want to close, just copy relevant line and paste and hit enter.

[developer@localhost:0 ~/work/notes]$ 


#####################################################################
To allow a port from firewall:
#####################################################################

iptables -A INPUT -p http --dport 3000 -j ACCEPT
#####################################################################
./deploy-workers.sh -a archive-node,admin-node,view-node -d marathon_ip=16.103.36.60 -d postgres_ip=16.103.36.60 -d imagetag=latest -- -Dvertica.host=16.103.35.155
./initjobservicedb.sh -h 16.103.36.60 -u postgres -p postgres


**************
To run jenkins docker instance
 docker run -p 8088:8088 -p 50000:50000 -v /root/jenkins_home:/var/jenkins_home -d jenkins
 (you shall have to create /root/jenkins_home directory; 8080 is port inside the container mapped to 8088 to host running the container)
**************

**************
Using sed to transform a pattern like ASP-0000 to http://some/link/ASP-0000

For this we need to remember the matched pattern. This is done by '&' variable. '&' remembers the matched pattern. A sample command can be 
like this:

%s/ASP-[0-9][0-9][0-9][0-9]/[&](https:\/\/jira.autonomy.com\/browse\/&)/

**************
