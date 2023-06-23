## Database Replication

## Master - Slave Replication.

### Install Mariadb on both Master and Slave.
```
apt install mariadb-server
```

### Start and enable mysql service on both master and slave.
```
systemctl start mysql
systemctl enable mysql
```

### Master Configuration.
Login to master and run below commands to edit mariadb server configuration file.
```
cd /etc/mysql/mariadb.conf.d
nano 50-server.cnf
```
Uncomment the below mentioned lines and replace 127.0.0.1 with Master IP.

```
bind-address            = 127.0.0.1
general_log_file       = /var/log/mysql/mysql.log
general_log            = 1
log_error = /var/log/mysql/error.log
server-id              = 1
log_bin
log_basename           = master
expire_logs_days       = 10
```

Restart mysql service. 
```
systemctl restart mysql
```


Using below command login to database with root password.
```
mysql -u root -p
```
Run the below command to create replica user and grant permission on Master Database.

**Note:** replace slave IP.

```
CREATE USER 'replica'@'<slave IP>' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'<slave IP>';
```

Run below command to lock Master Database.
```
FLUSH TABLES WITH READ LOCK;
```
Run the below command and store the output in notepad for future use.

**Note:** Need file value and position value while configuring on slave.
```
SHOW MASTER STATUS;
```
**Example output**
```
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| master1-bin.000005 |      344 |              |                  |
+--------------------+----------+--------------+------------------+
```

Exit from mysql CLI and run below command to dump the Master Database.
```
mysqldump -u root -p --all-databases --master-data > dbdump.sql
```
Login to mysql CLI and unlock tables.
```
mysql -u root -p
UNLOCK TABLES;
```

Using scp copy the dump from master to slave.
```
scp dbdump.sql slave_IP:/tmp
```

### Slave configuration.
Login to slave and run below commands to edit mariadb server configuration file.
```
cd /etc/mysql/mariadb.conf.d
nano 50-server.cnf
```
Uncomment the below mentioned lines and replace 127.0.0.1 with Slave IP.

```
bind-address            = 127.0.0.1
general_log_file       = /var/log/mysql/mysql.log
general_log            = 2
log_error = /var/log/mysql/error.log
server-id              = 1
log_bin
log_basename           = slave
expire_logs_days       = 10
```

Restart mysql service. 
```
systemctl restart mysql
```

Import mysql dump which we copied from Master.
```
mysql -u root -p < /tmp/dbdump.sql
```

Using below command login to database with root password.
```
mysql -u root -p
```

After login to mysql CLI run the below commands to update master information on Slave Database.

**Note:** Use the file and position value which we captured during configuration of Master.
```
STOP SLAVE;

CHANGE MASTER TO
MASTER_HOST='<Master IP>',
MASTER_USER='replica',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='master1-bin.000005',
MASTER_LOG_POS=344;

START SLAVE;
```

Using below command we can check the Slave status and configuration.
```
SHOW SLAVE STATUS\G
```
## Master - Slave Replication - Architecture.
![master-slave](https://github.com/vigneshkannan255/mariadb_replication/assets/32855922/7f4632a2-d193-4c30-8876-5ee24db80faf)


## Master - Master Replication.

## Login to slave and run below commands.
```
CREATE USER 'master'@'<Master IP>' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'master'@'<Master IP>';


STOP SLAVE;
```

Run the below command and store the output in notepad for future use.

**Note:** Need file value and position value while configuring on Master.
```
SHOW MASTER STATUS;
```
**Example output**
```
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| slave-bin.000001 |  2528082 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.000 sec)
```

## Login to Master and run below commands.
```
STOP SLAVE;

CHANGE MASTER TO
MASTER_HOST='Slave IP',
MASTER_USER='master',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='slave-bin.000001',
MASTER_LOG_POS=2528082;


START SLAVE;

SHOW SLAVE STATUS\G
```

## Master - Master - Replication  Architecture.
![master-master](https://github.com/vigneshkannan255/mariadb_replication/assets/32855922/8ed037b2-605c-4427-be2c-9802232551b7)


# Master - Master - Replication configured



