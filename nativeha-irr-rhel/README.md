# IBM MQ NativeHA on RHEL

---

# Table of Contents
- [1. Introduction](#introduction)
- [2. Workshop Environments ](#workshop-env)
- [3. Live Environment Setup](#live-setup)
  * [3a. Create Queue Manager ](#create-live-qm)
  * [3b. Create TLS Certificates](#tls-setup)
  * [3c. Update qm.ini](#update-live-qm-ini)
  * [3d. Start Queue Manager](#live-qmgr-start)
  * [3e. Disable Security](#disable-security)
  * [3f. Enable systemd Monitoring](#live-systemd)
- [4. Testing High Availability (HA) in Live Environment](#testing-live-ha)
  * [4a. Put and Get messages (amqsphac, amqsghac)](#ha-put-get)
  * [4b. Failover the Queue Manager](#ha-failover)
- [5. Summary ](#summary)

---
[Return to Main Menu](../index.md)
<br>

## 1. Introduction <a name="introduction"></a>

**What is Native HA?** <br>
Native HA is a high availability solution that is available on container deployments of IBM® MQ and on Linux.

**What is Native HA In-Region Replication (IRR)?** <br>
A Native HA In-Region Replication (IRR) configuration enables you to switch the running of a queue manager to a different Native HA configuration in a different location within the same region.

**In this lab**, you will investigate the process of configuring the NativeHA In-Region Replication (IRR) Queue Manager on RHEL Virtual Machines. Additionally, you will conduct testing in the Live environment, subsequently performing a failover to the Recovery environment and monitoring the transition of client connections from Live to Recovery.
.

![alt text](images/MQ-title-HA-VM.png)

<br>

## 2. Workshop Environments  <a name="workshop-env"></a>

You need to reserve Techzone environment which will have 6 RHEL VMs, and 1 WIndows VM. <br>
For this lab we will be using acemq1 and acemq4 for doing the MQ native HA IRR. <br>
We will launch everything from the Windows image.

Click on the Windows image console to open it.

![alt text](images/image.png)

<br>


## 3. Live Environment Setup <a name="live-setup"></a>

1. From the Windows console click on the **CAD** to get to the login page.  Click on OK for the Business Use Notice

   ![alt text](images/win1.png)

1. Login to the windows using techzone/IBMDem0s

   ![alt text](images/win2.png)

1. From the Windows VM's Console, open Putty program and open acemq1, acemq4 Virtual Machine sessions. <br>

   ![alt text](images/win3.png)


1. Arrange the windows on your desktop and you should have the 3 RH vms.    Login to each VM using ibmuser/engage.

   ![alt text](images/win4.png)
   

### 3a. acemq1 - Create Live Queue Manager <a name="create-live-qm"></a>

1. Run the following commands <br>

   Create Queue Manager MQ01HA <br>
   ```
   crtmqm -lr `hostname` -lf 8192 -lp 10 -ls 10 -p 1414 MQ01HA
   ```
   ![alt text](images/crtmq1.png)



### 3b. acemq4 - Create Recovery Queue Manager <a name="create-recovery-qm"></a>

1. Run the following commands <br>

   Create Queue Manager MQ01HA on the Recovery side <br>
   ```
   crtmqm -lr `hostname` -lf 8192 -lp 10 -ls 10 -p 1414 MQ01HA
   ```
   ![alt text](images/crtmq4.png)





### 3c. acemq1 - Create TLS Certificates <a name="tls-setup"></a>

1. Run the below steps to enable TLS on Queue Manager.  <br>
 
   Create TLS certificates. <br>
   ```
   runmqakm -keydb -create -db /var/mqm/qmgrs/MQ01HA/ssl/key.kdb -pw passw0rd -stash
   ```
   
   ```
   runmqakm -cert -create -db /var/mqm/qmgrs/MQ01HA/ssl/key.kdb -pw passw0rd -label selfsigned -dn CN=MQ01HA -size 2048
   ```
   
   ```
   sudo chown -R :mqm /var/mqm/qmgrs/MQ01HA/ssl/key.*
   ```
   ```
   sudo chmod g+r /var/mqm/qmgrs/MQ01HA/ssl/key.*
   ```

   ![alt text](images/crtmq1a.png)


2. acemq1 - Copy all key.* files to acemq4 (Recovery) Virtual Machines using sftp. 

   ```
   sftp ibmuser@acemq4
   ```
   ``` 
   mput /var/mqm/qmgrs/MQ01HA/ssl/key.* /var/mqm/qmgrs/MQ01HA/ssl
   ```
   ```
   quit
   ```

   ![alt text](images/crtmq1b.png)


4. acemq4 - Run the following commands. <br>
   ```
   sudo chown -R :mqm /var/mqm/qmgrs/MQ01HA/ssl/key.*
   ```
   ```
   sudo chmod g+r /var/mqm/qmgrs/MQ01HA/ssl/key.*
   ```
   ![alt text](images/crtmq1c.png)

   <br>

### 3c. acemq1 - Update Live qm.ini <a name="update-live-qm-ini"></a>

1. On VM acemq1, we will add the TLS parameters, NativeHARecoveryGroup stanza to qm.ini. 

   You can run the following command to look at the current **qm.ini** file.

   ```
   cat /var/mqm/qmgrs/MQ01HA/qm.ini
   ```
   ![alt text](images/qm1.png)

   
1. run the following commands. <br>

   ```
   cd ~/mqha-irr
   ```

   ```
   ./1-qm-ha.sh
   ```

   ![alt text](images/qm1a.png)



1. When done run the following command on acemq1 instance to verify that the **qm.ini** was updated correctly. 

   ```
   cat /var/mqm/qmgrs/MQ01HA/qm.ini
   ```

   ![alt text](images/qm1b.png)



### 3d. acemq4 - Update Recovery qm.ini <a name="update-recovery-qm-ini"></a>

   You can run the following command on **acemq4** to look at the current **qm.ini** file.

   ```
   cat /var/mqm/qmgrs/MQ01HA/qm.ini
   ```
   ![alt text](images/qm2.png)

   
1. We will now from the **acemq4** putty session, use vi and add below lines. <br>

```
./2-qm-IRR.sh
```


1. When done run the following command on acemq4 instance to verify that the **qm.ini** was updated correctly. 

```
cat /var/mqm/qmgrs/MQ01HA/qm.ini
```

![alt text](images/qm2b.png)



### 3d. Start Queue Manager <a name="live-qmgr-start"></a>

1. Run the following commands to restart the queue manager on acemq1,4. <br>

   ```
   strmqm MQ01HA
   ```
1. Once QMgr IS running ON acemq1, acemq4 run the following command. 

   The Queue Manager should be active in one of Virtual Machines. <br>

   ```
   dspmq -o nativeha -x
   ```
![alt text](images/qm3.png)


### 3e. Disable Security <a name="disable-security"></a>

1. Find the node that the QMgr is running as **acemq1** using this command.

   ```
   dspmq -o nativeha -x 
   ```

1. Run the following command, on the node where the queue manager is Active, <br>
This will disable security and define the channel and local Queue used for testing. 

```
runmqsc MQ01HA
ALTER QMGR CHLAUTH(DISABLED) CONNAUTH(' ')
REFRESH SECURITY TYPE(CONNAUTH)
DEFINE CHANNEL(NATIVEHACHL.SVRCONN) CHLTYPE(SVRCONN)
DEFINE QLOCAL(APPQ) DEFPSIST(YES)
```
<br>

![alt text](images/qm4.png)



### 3f. Enable systemd Monitoring  <a name="live-systemd"></a>

Reference: <br>
https://www.ibm.com/docs/en/ibm-mq/9.4.x?topic=ha-monitoring-restarting-ending-queue-manager-instances
<br>

You must implement a method to ensure that the queue manager instances in the Native HA configuration are still running, and restart them if required.<br>

Run the following commands on each RHEL VM. <br>

```
ln -s /opt/mqm/samp/mqmonitor@.service /etc/systemd/system 
````
```
sudo systemctl enable mqmonitor@MQ01HA
```
```
sudo systemctl start mqmonitor@MQ01HA
```
<br>


## 4. Testing High Availability (HA) in Live Environment <a name="testing-live-ha"></a>

### 4a. Put and Get messages (amqsphac, amqsghac)  <a name="ha-put-get"></a>

1. **On the Windows VM** <br> 
   start two command line programs. <br>
   
   ![alt text](cmdlines.png)

   On each Command Line window, run the following commands. <br>

   ```
   SET MQSERVER=NATIVEHACHL.SVRCONN/TCP/acemq1(1414),acemq4(1414)
   ```

   On acemq1, <br>
   ```
   amqsphac APPQ MQ01HA
   ```

   On acemq4, <br>
   ```
   amqsghac APPQ MQ01HA
   ```

   ![alt text](images/ha-test1.png)

<br>




## 4. Switching Roles  <a name="switch-roles"></a>

1. We will now check the status of both are Datacenter deployments.  If this is the first time you should see **Datacenter1 - Live** and **Datacenter2 - Recovery**

   You can run the script in either Datacenter.  In this example we are showing the command running in both Datacenters but you only need to run it on one of the putty instances.  

   ```
   ./get-status.sh 
   ```
   ![alt text](images/status1-CRR.png)


1. Run the following command from one of the putty instances and it will determine the current status and do the switch for each Datacenter as well as restart the QMgr on each instance.
   
   ```
   ./5-switch-crr.sh
   ```
    ![alt text](images/switch2-CRR.png)

   **Note:** Here you will see the switch script will determine which is Live and which is Recovery.   It will then switch the Roles and restart the QMgr. 
   <br>
   You will also observe that the putter and getter programs will reconnect to the new active instance of the QMgr.
   <br> 
1. You can now run the ./get-status.sh again to see current status and then the ./5-switch-crr.sh to switch roles back. 
 
   ```
   ./get-status.sh
   ```
     ```
   ./5-switch-crr.sh
   ```
  ![alt text](images/switch3-CRR.png)



## 5. Summary <a name="summary"></a>

Congratulations! At this point, you ought to be familiar with the process of configuring IBM MQ HA in a Primary region.

<br>
[Return to Main Menu](../index.md)


