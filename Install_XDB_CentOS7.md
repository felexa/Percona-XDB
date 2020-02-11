## Percona-XtraDB-Cluster
### Install instructions for CentOS 7 / RHEL7

### Assumptions
|Role|machine name|IP address|Operating System|
|-|-|-|-|
|master node 1|PXDB-1|10.10.10.1|CentOS 7.5|
|master node 2|PXDB-2|10.10.10.2|CentOS 7.5|

### On First node
##### Open firewall ports
```
firewall-cmd --permanent --add-port={3306/tcp,4444/tcp,4567/tcp,4568/tcp}
firewall-cmd --reload
```
##### Disable SELinux
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Add Percona Repository
```
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```
##### Install Percona-XtraDB-Cluster
```
yum install -y Percona-XtraDB-Cluster-57
```
##### Initial set up
```
systemctl start mysql
grep password /var/log/mysqld.log
mysql_secure_installation
systemctl stop mysql
```
##### Configure Replication Settings
```
cat >>/etc/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib64/galera3/libgalera_smm.so
wsrep_cluster_name=PXDB-cluster
wsrep_cluster_address=gcomm://10.10.10.1,10.10.10.2
wsrep_node_name=PXDB-1
wsrep_node_address=10.10.10.1
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=sstuser:IZFpWOS8DQlHfDFC
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Bootstrap/Initialize the Cluster
```
systemctl start mysql@bootstrap
```
##### Create Replication User
```
mysql -uroot -p -e "create user sstuser@localhost identified by 'IZFpWOS8DQlHfDFC'"
mysql -uroot -p -e "grant reload, replication client, process, lock tables on *.* to sstuser@localhost"
mysql -uroot -p -e "flush privileges"
```

### On Second node
##### Open Firewall ports
```
firewall-cmd --permanent --add-port={3306/tcp,4444/tcp,4567/tcp,4568/tcp}
firewall-cmd --reload
```
##### Disable SELinux
```
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Add Percona Repository
```
yum install -y http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm
```
##### Install Percona-XtraDB-Cluster
```
yum install -y Percona-XtraDB-Cluster-57
```
##### Configure Replication Settings
```
cat >>/etc/my.cnf<<EOF
[mysqld]
wsrep_provider=/usr/lib64/galera3/libgalera_smm.so
wsrep_cluster_name=PXDB-cluster
wsrep_cluster_address=gcomm://10.10.10.1,10.10.10.2
wsrep_node_name=PXDB-2
wsrep_node_address=10.10.10.2
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=sstuser:IZFpWOS8DQlHfDFC
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
EOF
```
##### Start mysql to join the cluster
```
systemctl start mysql
```
show status like 'wsrep%';

