MongoDB 6.0.6 Quick Start Guide for EXPRESSCLUSTER X on Windows
===

About this guide
---
This guide describes how to setup MongoDB 6.0.6 with EXPRESSCLUSTER X 5.1.

For the detailed information of EXPRESSCLUSTER X, please refer to [this site](https://www.nec.com/en/global/prod/expresscluster/index.html) .


## **System Overview**

### **System Requirement**
- Two servers
  - IP reachable each other
  - Having mirror disk
    - At least 2 partitions are required on each mirror disk.
    - Cluster partition size depends on ECX version.
    - X 5.1 or later: 1024MB
    - Data partition size depends on Database sizing.
- MongoDB Database are installed on local partition on each server.

### **System Configuration**
- Windows Server 2022 Standard
- MongoDB 6.0.6
- EXPRESSCLUSTER X 5.1

        Sample configuration
		<LAN>
		 |  +--------------------------+
		 |  | Primary Server           |
		 |  | - ecxcluster1 (Win2k22)  |
		 |  | - MongoDB 6.0.6         |
		 |  | - EXPRESSCLUSTER X 5.1   |
		 |  | IP Address:10.0.7.85     |
		 |  | RAM   : 6GB              |
		 |  | Disk 0: 50GB OS          |
		 |  |      C: local partition  |
		 |  | Disk 1: 30GB mirror disk |
		 |  |      E: cluster partition|
		 |  |      D: mirror partition |
		 |  +-----------+--------------+
		 |              |
		 |              | Mirroring
		 |              |
		 |  +-----------+--------------+
		 |  | Secondary Server         |
		 |  | - ecxcluster2 (Win2k22)  |
		 |  | - MongoDB 6.0.6         |
		 |  | - EXPRESSCLUSTER X 5.1   |
		 |  | IP Address:10.0.7.86     |
         |  | RAM   : 6GB              |
		 |  | Disk 0: 50GB OS          |
		 |  |      C: local partition  |
		 |  | Disk 1: 30GB mirror disk |
		 |  |      E: cluster partition|
		 |  |      D: mirror partition |
		 |  +--------------------------+
		 |

#### Cluster configuration
- Network Partition Resolution resource (PING method) : `pingnp1`

- failover Group: `mdbfailover`
	- Group resource
		- Fip                  : Floating IP address resource
		- md                   : mirror disk resource
		- mongodbservice       : service resource for MongoDB Server (MongoDB)

- Monitor resource

	- Fipw1             : Floating IP monitor resource
	- mdw1              : mirror disk monitor resource
	- servicew1         : service monitor resource for MongoDB Server (MongoDB)_service
	- userw             : user-mode monitor resource


## **Basic cluster setup on Primary and Secondary servers**


 ### **1. Install EXPRESSCLUSTER X (ECX)**
 ### **2. Register ECX licenses**
- EXPRESSCLUSTER X 5.1 for Windows
- EXPRESSCLUSTER X Replicator 5.1 for Windows

### **3. Create a ECX cluster with failover group On Primary server**
    - Network partition: 
        pingnp1: PING method
    - Group:
        MDBfailover
        Fip                  : Floating IP resource
        md                   : mirror disk resource
        
        
### **4. Start group on Primary server**

      
       +----------------------+-----------------------------+
       | Parameter            | Value                       |
       +----------------------+-----------------------------+
       | Name                 | MDBFailover                 |
       | Startup Server       | ecxcluster1 -> ecxcluster2  |
       | Floating IP Address  | 10.0.7.193                  |
       | Mirror Disk resource | D:\data                     |
       +----------------------+-----------------------------+

 If want to know how to add the resource, please refer to "EXPRESSCLUSTER X 5.1 for Windows Installation and Configuration Guide". 

 After you add failver group and execute apply the configuration file, you start failover group by ecxcluster1.  
     
### **5. Install MongoDB on both servers**

- Download and Install MongoDB 6.0.6 on both the servers.
    
    - For download procedure please refer to [this site](https://www.mongodb.com/try/download/community).
        - e.g.
        - mongodb-windows-x86_64-6.0.6-signed.msi

### Install MongoDB On both servers

1. Login as an user with Administrator privilege and Execute the .msi file on the server.
1. Installation Directory
    - Specify the local directory (C: drive) as **Installation Directory**.
    - e.g. *C:\Program Files\MongoDB\Server\6.0*
1. Choose satup type
    - Complete
1. Data Directory
    - Specify the local directory (C: drive) as **Data Directory**.
    - e.g. *C:\Program Files\MongoDB\Server\6.0\data*
1. Ready to Install
    - Click **Next**
1. Completing the MongoDB Setup Wizard
    - Uncheck the check box and click **Finish**
    - You need to install MongoDB Compass to connect MongoDB database, please keep it checked and click Finish.

### **6. Configure MongoDB Database in Mirror drive (Node1)**

- Create the database directory in mirror drive e.g. D:\data
- Stop MongoDB service from services.msc.
- Copy MongoDB data from default directory (C:\Program Files\MongoDB\Server\6.0\data) to Mirror disk (D:\data)
- Right Click on the newly created folder and make sure that it has all type permissions for local MongoDB system user.

### **7. Configure MongoDB Configuration file on both Nodes.**

- Open the configuration file in a text editor and modify the MongoDB configuration file with storage location.
- The MongoDB configuration file (mongod.cfg) is located at: (C:\ProgramFiles\MongoDB\Server\6.0\bin) in windows.
```
storage:
     dbPath: D:\data
```
- Save the changes in mongod.cfg and start MongoDB service from services.msc.


### **8. Change the startup type of MongoDB Service and Configure MongoDB Service in ECX WebUI(Node1 & Node2)**

- Open the Windows Service Manager     
  - On Command Prompt
        > services.msc
- Change the startup type of MongoDB Service to Manual.
- Add the service resource to control MongoDB and configure in Config mode of the Cluster WebUI.
- mongodbservice       : service resource for MongoDB Server(MongoDB)
- Execute Apply the Configuration File in Config mode of the Cluster WebUI.

          
### **9. Testing Of MongoDB Database on ECX Cluster**

- Open MongoDB Compass and connect with mongodb://localhost:27017 on Node1.

Open MONGOSH shell where default database is Test.
To list the databases available to the user, use the helper command "show databases".     
```
    Test> show databases;
    admin    40.00 KiB
    config  112.00 KiB
    local    72.00 KiB

```

- Create a new database by using command **use (DB)** with the database that you would like to create.
For example, the following commands create both the database **Demo1** and the collection
 **myCollection** using the **insertOne()** operation:

```
    Test> use Demo1
    <switched to db Demo1

    Demo1>db.myCollection.insertOne( { x: 1 } );
    <{
    acknowledged: true,
    insertedId: ObjectId("649aaf4cdb37b65196c5f832")
    }
    >db
    <Demo1

    Demo1>show databases;
        Demo1    40.00 KiB
        admin    40.00 KiB
        config  108.00 KiB
        local    72.00 KiB

```  
### **10. After moving group failover on Node 2 to check newly created Database " Demo1" for testing MongoDB database.**

- Check and Confirm the database on MongoDB which we have created on node1 with the help of MongoDB Compass tools.

```
    Demo1>show databases;
        Demo1    40.00 KiB
        admin    40.00 KiB
        config  108.00 KiB
        local    72.00 KiB
```


- The data is replicated from primary to secondary server successfully. 

### **11. Verification**

- Confirm that we can access the database where it failover group is running.

