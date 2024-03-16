# mysql
mysql replication

**Install**
```
yum install -y wget
wget https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
rpm -Uvh mysql80-community-release-el7-6.noarch.rpm
yum -y install mysql-server mysql-client
mkdir -p /var/log/mysql
------------------------
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-el7-6.noarch.rpm
yum install -y mysql-community-server
mkdir -p /var/log/mysql
```
**Initialize Server**
```
mysqld --initialize
chown -R mysql: /var/lib/mysql
chown -R mysql: /var/log/mysql
vi /etc/my.cnf
```
**Change config**
```
[mysqld]
bind-address=0.0.0.0
server-id=1
log_bin=/var/log/mysql/mybin.log

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```
**Users**
```
mysql -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Pass00word!';
FLUSH PRIVILEGES;

CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY ‘12345678’;
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
```
**Master Status**
```
SHOW MASTER STATUS;
!!! Важно запомнить File и Position

If database exists lock tables before dump
FLUSH TABLES WITH READ LOCK;
then dump the database
mysqldump -u root -p databasename > databasename.sql
UNLOCK TABLES;
Or dump db with row binlog coordinates
mysqldump --all-databases --master-data > /backup/path/dbdump.db
After create database on slave
restore backup to slave
mysql -u root -p databasename < databasename.sql
```
**Replica On Slave**
```
CHANGE MASTER TO MASTER_HOST='192.168.1.152', MASTER_USER='replication', MASTER_PASSWORD='Repl11Pass!', MASTER_LOG_FILE = 'mybin.000001', MASTER_LOG_POS = 1161;
START SLAVE;
SHOW SLAVE STATUS\G;
```
- Slave_IO_State, Slave_SQL_State — состояние IO потока, принимающего двоичный журнал с мастера, и состояние потока, применяющего журнал ретрансляции соотвественно. Только наличие обоих потоков свидетельствует об успешном процессе репликации.
- Read_Master_Log_Pos — последняя позиция, прочитанная из журнала мастера.
- Relay_Master_Log_File — текущий файл журнала мастера.
- Seconds_Behind_Master — отставание слейва от мастера, в секундах.
- Last_IO_Error, Last_SQL_Error — ошибки репликации, если они есть.
