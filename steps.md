1. download docker image mysql:5.7
2. create master/slave conf/data folder 

    2. need to modify the container name.
    3. the mysql root user is created during start up process with password.
3. create cnf/mysql.cnf file as configuration for master and slave db as folder presents.
4. start up the master container
    * need to modify the image id in run-master.sh
5. login to docker console and configure 'reader' user
 > [root@localhost ~]# docker exec -it 6405439bdd40 /bin/bash //注意修改容器ID
 > root@6405439bdd40:/# mysql -u root -pmasterpwd 
 > 
 > mysql> GRANT REPLICATION SLAVE ON *.* to 'reader'@'%' identified by 'readerpwd';
 > Query OK, 0 rows affected, 1 warning (1.60 sec)
 > 
 > mysql> FLUSH PRIVILEGES;

6. create and run slave db container with 'run-slave.sh'
7. log on to master db container and check master status
 > mysql> show master status;
 >   +------------------+----------+--------------+------------------+-------------------+
  >  | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
 >   +------------------+----------+--------------+------------------+-------------------+
 >   |** mysql-bin.000003** |     **765** |              |                  |                   |
 >   +------------------+----------+--------------+------------------+-------------------+
 >   1 row in set (0.01 sec)

8. logon to slave db container add maste access point:
[root@localhost ~]# docker exec -it fbc7e934f424 /bin/bash
root@fbc7e934f424:/# mysql -u root -pslavepwd
 > mysql> change master to master_host='172.17.0.2',master_user='reader',master_password='readerpwd',master_log_file='mysql-bin.000003',master_log_pos=765;
 > 
 > find master container ip info with **'cat /etc/hosts'**  command

9. start up slave process in slave db container
 >mysql> start slave;
 > Query OK, 0 rows affected (0.03 sec)
 > mysql> show slave status\G
 >          Slave_IO_State: Waiting for master to send event
 >             Master_Host: 172.17.0.2
 >             Master_User: reader
 >             Master_Port: 3306
 >           Connect_Retry: 60
 >         Master_Log_File: mysql-bin.000003
 >     Read_Master_Log_Pos: 765
 >          Relay_Log_File: b3a8ba2fdc0c-relay-bin.000002
 >           Relay_Log_Pos: 494
 >   Relay_Master_Log_File: mysql-bin.000003
 >    **    Slave_IO_Running: Yes **
 >    ** Slave_SQL_Running: Yes **
 
 >  ** as long as ‘Slave_IO_RUNNING' and 'Slave_SQL_Running' is 'Yes', it is working.

10. Verify.
    1. create database in master container
    > mysql> create database slavetest;
    > mysql> show databases;
 > +--------------------+
 > | Database           |
 > +--------------------+
 > | information_schema |
 > | mysql              |
 > | performance_schema |
 > | **slavetest**          |
 > | sys                |
 > +--------------------+

   2. check databases in slave container
 > show databases;
 > ......
 > | slavetest      |
 > ......
