# Migrate data between two databases Using AWS Data Migration Service

## Aim :
* To migrate data from Aurora MySQL Source database to DynamoDB target database using Data Migration Service in AWS Cloud.

![DMSArchitecture](https://drive.google.com/uc?export=view&id=1FrbOvcNQ5kaqLwm3-JH1BJBuAW_f7UDN)

## Steps to be followed for a successful data migration :
1. Create RDS Aurora MySQL Database
2. Create DMS Replication Instance
3. Create Source and Target Endpoint
4. Create Parameter Group
5. Create Bastion Host to connect RDS Database
6. Workbench Login and Create Database
7. Create Data Migration Task

## Step 1. Create Source RDS
* I'll be choosing Aurora MySQL for this project. You can choose any database.

![SourceDatabase](https://drive.google.com/uc?export=view&id=1HcX-jta-Pkvxsh-Q42w3S6C1wtqYLbcW)
* Provide username and password, only then you can login to your MySQL Database later from other resources like MySQL Workbench.

![UsernamePasswd](https://drive.google.com/uc?export=view&id=1VD0rCZIO6gXFXZDAj_YcQvgIN3e6ICZq)

* At Instance Configuration Section Choose db instance class as ```db.t3.small``` to reduce your cost.
* Network type default VPC and subnet group and you enable or disable public access. If you provide public access, then your db can be accessed from outside VPC.
* Create new security group and check the port ```3306``` is enabled

![SecurityGroup](https://drive.google.com/uc?export=view&id=1eTKzZjHOzf4hLv-uZsgEhSKGTFeVsTqd)
* After configuring click create database. It will take some time to create.

## Step 2. Create DMS Replication Instance
* Create a Data Migration Replication Instance to migrate from source to target. Go to DMS and under replication instance you can create configure.

![DMSRepInstance](https://drive.google.com/uc?export=view&id=17G6tRhbArOpm6JVjHhPZd4YDqyXhMGey)

![RepInstanceConfig](https://drive.google.com/uc?export=view&id=1r2cySaAVxAzQYP4R7sFJc2WVKBRrUgB0)

* Also config connectivity network and public access, then create the replication instance. It take 10-15 mins to create.

* After created attach the IP Address to the inbound rule in RDS Security Group which we created newly ```aurora-mysql-sg``` only then the connectivity will be enabled.

![RepInstanceCreated](https://drive.google.com/uc?export=view&id=1EIkHBLy9pCNlpaKTDJK8bvV951cl1C7E)

![AttachInboundRule](https://drive.google.com/uc?export=view&id=1fZvwZZ_8-gHriOAv3k1IxBTohH_rO8hY)

## Step 3. Create Source and Target Endpoint
* First we have to Create Source endpoint, which connects Source MySQL to DMS.

![CreateEndpoint](https://drive.google.com/uc?export=view&id=10KHzczLGKiWk8IsRplFCFXpqW3ZDQCbn)

![ConifSourceEndpoint](https://drive.google.com/uc?export=view&id=1NPyEPTw66YIFQ4jN49hqwrLFwoaZLKWb)

* Run Test Connection, if it result in successful then all the configuration till are done correctly.

![SourceRunTest](https://drive.google.com/uc?export=view&id=1wztWgI6a2Mg6hEJ2ThAcpShZsKfO_PCO)

* Now We are going to create target endpoint, which will connect DMS to Dynamodb. For this we need to create ```IAM Role```
[IAM Role For Dyanmo Db as target](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.DynamoDB.html). Configure it with account id and region. If it still confuses you then just enable ```administrator-access``` policy while creating DMS-Dynamodb Role.

* Create Target Endpoint

![TargetEndpoint](https://drive.google.com/uc?export=view&id=1X2DoBRxZhJKH3drMzoKhoddGcI4PeUi8)

![RunTestConnection](https://drive.google.com/uc?export=view&id=1-IoTh8DfBEL6isy3FgT3H7WIbRwCJ5kY)

* Now both source and target endpoints are created successfully.

## Step 4. Create Paramter Group on RDS.
* Create Paramter group for binlogs and attach it to the RDS Database Instance.

![CreateParameter](https://drive.google.com/uc?export=view&id=1MituOT9Q609UvyhIThF_9wptRirJhzzE)

* After created, Modify binlog checksum and binlog format by clicking Edit.

![ModifyParameterGroup](https://drive.google.com/uc?export=view&id=1IY0KlCpbmb8ioKpLPQFn3QTR3tiNp0y3)

* After modification, click save changes and attach it to MySQl Rds by clicking modify.

## Step 5. Create Bastion Host
* To connect to RDS Database we need to create EC2 Machine as a Bastion Host. Using this machine we can connect MySQl Database.

![Ec2Machine](https://drive.google.com/uc?export=view&id=1TOb03wppgOwttGqTWJ4mtaMQ7c0hUPzs)

* Create New Security group which has EC2 Machine's Private IP Address as a inbound rule. Attach that security group to RDS instance. So now the connection will be enabled between EC2 instance and MySQl Database Instance.

![BastionHostSG](https://drive.google.com/uc?export=view&id=1tVY7rUJpkCNVvjI9AS-6nBd8wB3NEt9Y)

## Step 6. Login to MySQL using Workbench
* Before login to Db in workbench we need public ip address of EC2 Machine, MySQL Db's endpoint which is available in rds database.

![WorkbenchConfig](https://drive.google.com/uc?export=view&id=1MQg6cAQlj7CuH4gB-4vIZTCNP70Khxzm)

* Click ok and it will ask for password again. Provide password and now the setup is ready.

![WorkbenchLogin](https://drive.google.com/uc?export=view&id=1if6v2eXJVl1Gtztjv2bsnTgPpwVyuAJf)

* Execute below queries and check

```
show schemas;
```
```
call mysql.rds_set_configuration('binlog retention hours', 24);
SHOW VARIABLES LIKE 'log_bin';
```
![bin_logs-off](https://drive.google.com/uc?export=view&id=1HcG0E4HpBApPz1QUZ2Qu_M2mYC58pjWr)

* If it shows log_bin off then, Reboot your RDS Database Instance.

![RebootRDS](https://drive.google.com/uc?export=view&id=1NwiByXZ-_5Wcxw0ROF2VLxYPLDcabdGm)

* After RDS instance available, execute same query on workbench. it gives ouput as ON.

![log_bin-ON](https://drive.google.com/uc?export=view&id=1EBJVkXQUCmxim6RE30K8ocIg5-i1M6Hh)

* Execute some basic SQL queries to Create Schema ```mySchema``` and table ```employee``` or execute large workload query as your wish.

```
create schema if not exists mySchema;

use mySchema;

create table if not exists employee(
   emp_id int auto_increment primary key,
   emp_name varchar(100)
);

insert into employee values(1,'Mari');
insert into employee values(2,'Shelby');
insert into employee values(3,'Arthur');
insert into employee values(4,'John');

select * from employee;
```

![SqlQuery](https://drive.google.com/uc?export=view&id=1KMlyfaGzajahK6pHST-GJYVxv4yTRpxl)

* MySQL Database is ready and it's time to migrate our data to dynamodb.

## Step 7. Create Migration Task.
* On DMS under migration task create a new migration task. Name the task ```mysql-dynamodb``` or ```mysql-dynamodb-task``` or whatever, select replication instance, source and target endpoint, select ```migrate existing data and replicate ongoing changes```.

![CreateMigrationTask](https://drive.google.com/uc?export=view&id=1N-x7fgCM62-XiOpqhQRpNCkpwo3LYdTU)

* You can config partcicular schema and table. 

![ConfigTask](https://drive.google.com/uc?export=view&id=1vd3E3wcjzhm_uXCvWueC32_iGJ7siNWm)

* It will create, run and complete the task.

![CreatingTask](https://drive.google.com/uc?export=view&id=1lcue2EXZXIM_CfZFTNSKTw9eHFvKKmSl)

![TaskRunning](https://drive.google.com/uc?export=view&id=12WpXI-gS5JW2t1yTNPt_fnQMmannL6Er)

![TaskCompleted](https://drive.google.com/uc?export=view&id=1P5r80pzAiQvNPt6FLcGGCVthJ0qPccBL)

* Check the task overview for successful completion

![TaskOverview](https://drive.google.com/uc?export=view&id=1weKTKFgc6BYC6TavdOPPnAlFzoK6O5a3)

## Step 8. Check Dynamodb Database
* Now go to Dynamodb and check tables. New tables will be created.

![DynamodbTables](https://drive.google.com/uc?export=view&id=1qZAlVgaKGwHn0Im3p9emxmW26LSzJONN)

* select employee and go inside and check to the items.

![ViewItems](https://drive.google.com/uc?export=view&id=1lZnI5Vlifm2PY4Rn05i9VuirZ3vSzZAp)

* Yesss!!! The data from MySQL migrated successfully to dynamodb.

## FAQ
### 1. Will aws charge us for this project ?
Yes. RDS Database instance and DMS replication instance is not free.
So don't forget to delete it after completion.
First delete migration task, endpoints, replication instance and rds instance later EC2 instance.

### 2. Can i migrate from MySQL to S3 ?
Yes. You need an IAM Role ```dms-s3-role``` attach policy [S3 as a Target](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.S3.html) or you can attach only ```administrator-access``` policy for demo purpose. While creating target endpoint to s3 you need to provide that IAM role's ARN.

### 3. Why error occurs ?
If the inbound rule in security group misconfigured then it will cause error during run test connection in target endpoint.

Check both RDS instance and DMS replication instance are in same VPC.

EC2 Instance's private ip address must be attached with RDS Instance security group. Then note down public ip address of EC2 instance, rds instance endpoint, pem file, database's username password to login MySQL properly.

If you're migrating data cross accounts then you should enable VPC peering.