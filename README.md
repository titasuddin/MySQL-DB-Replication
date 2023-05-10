# How to create MySQL Master Master replication
Mysql RnD

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
