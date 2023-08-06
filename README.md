# How to create MySQL Master Master replication
Mysql DB repllication


 In this tutorial, we will explore the process of setting up Master-Master MySQL Database Replication. This replication method,
also referred to as "mirror," allows for real-time updates between both servers to maintain the latest version of data.
Although the initial setup may be expensive, the benefits include increased redundancy, availability, and application uptime.


Prerequisites:
1. Two Server with SSH access with sudo previleges,we took Ubuntu 22.04 LTS Server and name those Master1 and Master2
  
2. Same version of MySQL, we took MySQL 8.0

Step 1: Install MySQL both server Master1 and Master2 and configure same as below:

      $ sudo apt-get update
      $ sudo apt-get install mysql-server mysql-client -y
      $ sudo systemctl status mysql.service
      $ sudo mysql
      mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'My8SQL$pass';
      mysql> \q
      
      
      
Step 2: Edit msyql configuration file and update following parameter as same on both server 
      
      $ sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf and edit as same
        #bind-address           = 127.0.0.1
        #mysqlx-bind-address    = 127.0.0.1
        server-id               = 1 # put value 2 in Master2 server
        log_bin                 = /var/log/mysql/mysql-bin.log
      $ sudo systemctl restart mysql.service
      $ sudo systemctl status mysql.service
      
      
Step 3: Create the Replicator User(s) on both server as same below
      
      $ sudo mysql -u root -p then enter password
      mysql> create user 'rep_user'@'%' identified by 'rep_Pass#1';
      msyql> alter user 'rep_user'@'%' identified with mysql_native_password by 'rep_Pass#1';
      msyql> grant replication slave on *.* to 'rep_user'@'%';
      mysql> flush privileges;

Step 4. Configure Replication from Master1 to Master2

      //run the following on Master2 server and note the File and position
      
      mysql> show master status;
      
          +------------------+----------+--------------+------------------+-------------------+
          | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
          +------------------+----------+--------------+------------------+-------------------+
          | mysql-bin.000001 |     1151 |              |                  |                   |
          +------------------+----------+--------------+------------------+-------------------+
      
      //we have run the the following command on Master1
      //update actual value of Master2 before run
      
      CHANGE MASTER TO MASTER_HOST='IP_of_Master_Server_2',
      MASTER_USER='MySQL_user_of_the_Master_Server',
      MASTER_PASSWORD='Replication_user_password',
      MASTER_LOG_FILE='Log_on_Master_Server_2',
      MASTER_LOG_POS=Position_of_log_file_on_Master_Server_2;
      
      // in my case
      
      mysql> STOP SLAVE;
      mysql> CHANGE MASTER TO master_host='65.2.92.230', master_port=3306, master_user='rep_user', master_password='rep_Pass#1', master_log_file='mysql-bin.000001', master_log_pos=1151;
      mysql> START SLAVE;

      
Step 4. Configure Replication from Master2 to Master1     
      
//run the following on Master1 server and note the File and position
      
      mysql> show master status;
      
          +------------------+----------+--------------+------------------+-------------------+
          | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
          +------------------+----------+--------------+------------------+-------------------+
          | mysql-bin.000001 |     1152 |              |                  |                   |
          +------------------+----------+--------------+------------------+-------------------+
      
      //we have run the the following command on Master2
      //update actual value of Master1 before run
      
      CHANGE MASTER TO MASTER_HOST='IP_of_Master_Server_1',
      MASTER_USER='MySQL_user_of_the_Master_Server',
      MASTER_PASSWORD='Replication_user_password',
      MASTER_LOG_FILE='Log_on_Master_Server_1',
      MASTER_LOG_POS=Position_of_log_file_on_Master_Server_1;
      
      // in my case
      
      mysql> STOP SLAVE;
      mysql> CHANGE MASTER TO master_host='35.169.157.171', master_port=3306, master_user='rep_user', master_password='rep_Pass#1', master_log_file='mysql-bin.000001', master_log_pos=1152;
      mysql> START SLAVE;
      
Step 5: Testing the replication
      //Go to Master2 server and login to mysql console 
      $ sudo mysql -u root -p
      mysql> show databases;
      
          +--------------------+
          | Database           |
          +--------------------+
          | information_schema |
          | mysql              |
          | performance_schema |
          | sys                |
          +--------------------+
      
      msyql> create database demo;
      
      // Now check run the following on Master1
      
      mysql> show databases;
          +--------------------+
          | Database           |
          +--------------------+
          | demo               |
          | information_schema |
          | mysql              |
          | performance_schema |
          | sys                |
          +--------------------+
      
      // New database is also here and Now will create a table & insert some value from Master1
      
      msyql> use demo;
      msyql> CREATE TABLE authors (id INT, name VARCHAR(20), email VARCHAR(20));
      msyql> show tables;
          +----------------+
          | Tables_in_demo |
          +----------------+
          | authors        |
          +----------------+
      msyql> INSERT INTO authors (id,name,email) VALUES(1,"Vivek","xuz@abc.com");
      
      // Now goto Master2 server and check new data
      
      mysql> use demo;
      mysql> SELECT * FROM authors;
          +------+-------+-------------+
          | id   | name  | email       |
          +------+-------+-------------+
          |    1 | Vivek | xuz@abc.com |
          +------+-------+-------------+
          1 row in set (0.00 sec)
      // here we see new data already replicated to Master2 server


You are done !! Thank you !!
