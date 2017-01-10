# Welcome to my playground! :)

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

# Table of Contents
  * **Windows Platform**
    * [[Adding a system to a domain]]
    * **Exchange/Active Directory**
      * [[Adding User to AD and allowing RDP]]
      * [[Exchange Transport Service wont start]]
      * [[Autodiscovery, Exchange: Connectivity Issues]]
      * [[AD DC More notes]]
    * [[Creating a network sharing folder in windows]]
    * [[Working with powershell]]
    * [[Vmware internet not working issue]]
    * [[MSSQL Random Notes]]
  * **Linux**
    * **Bash**
      * [[Alias having quotes in command]]
    * [[Journalctl utility]]
    * [[Find and regex]]
    * [[docker/windows mount/iptables/sed/jenkins]]
    * [[NFS (Network File System) server]]
    * [[Postfix commands]]
    * **Mesos**
      * [[Mesos Cluster Setup (Master and Slaves different machines)]]
      * [[Mesos Marathon Docker deployment issues]]
    * [[yum update error]]
  * **MSSQL/SQL/Postgres**
    * [[Setting up postgres replication using pgpool]]
    * [[SQL]]
    * [[MSSQL Random Notes]]
  * **Work Project Specific**
    * [[Deploy Commands]]
