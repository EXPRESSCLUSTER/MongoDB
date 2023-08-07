MongoDB with EXPRESSCLUSTER X on Linux
===

About this guide
---
This guide describes how to setup MongoDB with EXPRESSCLUSTER X. 
For the detailed information of EXPRESSCLUSTER X, please refer to [this site](https://www.nec.com/en/global/prod/expresscluster/index.html) .


Configuration
---
In this setup, create 2 nodes (Node1 and Node2 as below) mirror disk type cluster.
Achieving MongoDB high availability By using EXPRESSCLUSTER X. 

### Software Versions
- MongoDB 6.0 (internal version:6.0.4)            
- EXPRESSCLUSTER X 5.1 for Linux 
- EXPRESSCLUSTER X Replicator for Linux

### Cluster Configurations
- Group resources
  - Exec resource
  - Floating IP resource
  - Mirror disk resource
- Monitor resources
  - Floating IP monitor resource
  - Mirror disk connect monitor resource
  - Mirror disk monitor resource
  - MongoDB custom monitor resource

MongoDB Setup
---
Please note that the following points are different if you set mongoDB to EXPRESSCLUSTER.
- Database have to create Mirror disk that managed by EXPRESSCLUSTER.
  You have to set only active server if you create database and database cluster.


Procedure
---
1. EXPRESSCLUSTER setup  

- We assume the following 2node cluster and explain it.

    ### Cluster Information
    ||Node1(Active)|Node2(Standby)|
    |---|---|---|
    |Server name|Server1|Server2|
    |IPaddress|10.0.7.174|10.0.7.175|  
    |Cluster partition|/dev/sdb1|/dev/sdb1|
    |Data partition|/dev/sdc1|/dev/sdc1|
  
    ### Failover Group Information  
    |parameter|value|
    |---|---|
    |Name|failover1|
    |Startup Server| Server1 -> Server2 |
    |Floating ip address|10.0.7.176|
    |Mirror disk resource (mount point)|/mnt/md1|
    
    - In Config mode of the Cluster WebUI, add failover group to use mongoDB.  
      You need the following resources.
      - Floating ip resource  
      - Mirror disk resource
    
     If want to know how to add the resource, please refer to "EXPRESSCLUSTER X 4.0/1/2/3 for Linux Installation and Configuration Guide". 
     After you add failover group and execute apply the configuration file, you start failover group by server1.  
     
2. Install mongoDB on both servers

    - Configure the package management system (yum).
      
      Create a **/etc/yum.repos.d/mongodb-org-6.0.repo** file so that you can install MongoDB directly using yum:

   ```
      [mongodb-org-6.0]  
      name=MongoDB Repository
      baseurl=https://repo.mongodb.org/yum/redhat/$releaseve/mongodb-org/6.0/x86_64/  
      gpgcheck=1   
      enabled=1
      gpgkey=https://www.mongodb.org/static/pgp/server-6.0.asc

    ```
      
    - Install the MongoDB packages using Yum
      1. yum install -y mongodb-org
      
            
    OR  
       
      Note :- Please visit [this site](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-red-hat/) if any problems arise with the installation and setup of mongoDB

    
        
3. MongoDB Configuration for Mirror disk (Node1)

    - Create the database directory.
        ```
        # mkdir -p /mnt/md1/mongo
        ``` 

    - Copying mongoDB data from default location to Mirror Disk.
      
      ```
      # systemctl status mongod.service
      # systemctl stop mongod.service
      # systemctl disable mongod.service
      # sudo rsync -av /var/lib/mongo /mnt/md1/mongo
      # chown -R mongod:mongod /mnt/md1/mongo
      ```
4. Perform the below steps on both the Nodes.

    - Initialize the mongoDB 6.0(internal version:6.0.4)
        
    - Edit the configuration file  and modify the following fields accordingly: (/etc/mongod.conf).

         > storage.dbPath to specify a new data directory path (e.g. /mnt/md1/mongo)

         > systemLog.path to specify a new log file path (e.g. /some/log/directory/mongod.log)

        
              
5. MongoDB Setup (Node1)
        
    - Run the mongoDB service.
        ```
        # systemctl start mongod.service
        ```
    - login into mongodb 
        ```
        # mongosh    
        ```
    - Create database named db_test.
        ```
        -> use db_test
        ```        
    - Create Collection
        ```     
        -> db.myCollection.insertOne( { x: 1 } );

        -> show dbs
            admin   40.00 KiB
            config  12.00 KiB
            db_test 40.00 KiB
            local   80.00 KiB
        
        -> exit
        ```
    - Stop mongoDB service.
        ```
        # systemctl stop mongod.service
        ```   

6. MongoDB Setup (Node2)

    You have to move failover group on the Node2. Configure mongoDB on the Node2.
    - Start mongoDB service.
        ```
        # systemctl start mongod.service
        ```
    - login into mongodb
        ```
        # mongosh    
        ```
    - Check created database

        ```     
        -> show dbs
            admin   40.00 KiB
            config  12.00 KiB
            db_test 40.00 KiB
            local   80.00 KiB

        -> exit
        ```  
              
    - Stop mongoDB service.
        ```
        # systemctl stop mongod.service
        ```    
 7. Configure the EXPRESSCLUSTER
 
      - Add the exec resource and configure. 
       
        In Config mode of the Cluster WebUI, Add the exec resource to control mongoDB.  
          - Configure the start.sh and stop.sh
            -  In the case of start.sh -> Immediately after "$CLP_DISK" = "SUCCESS", add the "systemctl start mongod.service"
            -  In the case of stop.sh  -> Immediately after "$CLP_DISK" = "SUCCESS", add the "systemctl stop mongod.service"
           
      - Add the mongoDB custom monitor resource
          - Configure the following parameters

              |parameter|value|
              |---|---|
              |Monitor(common) > Monitor Timing > Target Resource|setting the exec resource name|
              |Monitor(special) > script created with this product|genw.sh*|
              |Monitor(special) > Monitor type > |synchronous|
              |Monitor(special) > Log Output Path > |/opt/nec/clusterpro/log/mongod|
              |Recovery Action > Executing failover to the recovery target|failover1|

             ```
              #! /bin/sh
              name=mongod.service
              systemctl status $name
              ret=$?
              if [ $ret = 0 ]; then
              echo $name is running.
              exit 0
              else
              echo $name is NOT running.
              clplogcmd -m "$name is NOT running."
              exit 1
              fi
              ```
              
      - In Config mode of the Cluster WebUI, execute Apply the Configuration File.
      

8. Verification

    - Confirm that we can access the database where it failover group is running.
