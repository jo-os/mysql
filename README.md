# mysql
mysql replication
Install
```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
yum install -y mysql-community-server
mkdir -p /var/log/mysql
```
Initialize
```
mysqld —initialize
chown -R mysql: /var/lib/mysql
chown -R mysql: /var/log/mysql
vi /etc/my.cnf
```
Change config
```
[mysqld]
bind-address=0.0.0.0
server-id=1
log_bin=/var/log/mysql/mybin.log
```
Users
```
mysql -p
ALTER USER ‘root’@‘localhost IDENTIFIED BY ‘123456’;
FLUSH PRIVILEGES;

CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY ‘12345678’;
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
```
Master Status
```
SHOW MASTER STATUS;
!!! Важно запомнить File и Position
```
Replica
```
CHANGE MASTER TO MASTER_HOST='192.168.10.17', MASTER_USER='replication', MASTER_PASSWORD='1234
5678', MASTER_LOG_FILE='mybin.000001', MASTER_LOG_POS=1162;
START SLAVE;
SHOW SLAVE STATUS\G;
```
