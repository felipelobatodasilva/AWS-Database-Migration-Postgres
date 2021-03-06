# AWS-Database-Migration-Postgres

For this migration, We'll use AWS Database Migration Service (DMS) and many other Services like IAM, EC2, S3 and RDS. 

All of these services mentioned recently can be used under AWS Free Tier which offers some different services from AWS for free, in order to explore many of them.

# Table of Contents
● [Project Overview](#overview)<br/>
● [Preparing The Environment For The Migration](#preparingenvironment)<br/>
&emsp;◌ [Creating S3 Bucket](#creatings3bucket)<br/>
&emsp;◌ [Creating Postgres RDS](#creatingRDS)<br/>
&emsp;◌ [Launching an EC2 instance](#launchingEC2)<br/>
&emsp;◌ [Connecting RDS locally with PGAdmin and Setting up Database](#connectingRDS)<br/>
<!--&emsp;◌ [Connecting EC2 instance](#connectingEC2)<br/>-->

● [Migrating Data from RDS DB instance to S3](#migratingdatafromRDS)<br/>
&emsp;◌ [Creating IAM Role for S3 and RDS](#creatingIAMrole)<br/>
&emsp;◌ [Creating a Replication Instance on DMS and Endpoints](#creatingreplication)<br/>
&emsp;◌ [Run The Task For Migration](#formigration)<br/>

## Project Overview <a name="overview"></a>

For this project, we're gonna use EC2, S3 and RDS as follow with their respective brief explanation, in order to undertand a little bit about what each of them can do and why is used for

[__AWS EC2__](https://aws.amazon.com/ec2/?ec2-whats-new.sort-by=item.additionalFields.postDateTime&ec2-whats-new.sort-order=desc) -> Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers. Amazon EC2’s simple web service interface allows you to obtain and configure capacity with minimal friction. It provides you with complete control of your computing resources and lets you run on Amazon’s proven computing environment.

[__AWS DMS__](https://searchaws.techtarget.com/definition/AWS-Database-Migration-Service-AWS-DMS) -> AWS Database Migration Service (DMS) is a software tool for migrating an on-premises database to the Amazon Web Services cloud. The service aims to reduce the duration of database transfers, which can take months otherwise. AWS DMS can handle homogenous migrations, such as Oracle to Oracle, as well as heterogeneous migrations, such as Oracle to MySQL.

[__AWS RDS__](https://searchaws.techtarget.com/definition/Amazon-Relational-Database-Service-RDS) -> Amazon Relational Database Service (RDS) is a managed SQL database service provided by Amazon Web Services (AWS). Amazon RDS supports an array of database engines to store and organize data. It also helps with relational database management tasks, such as data migration, backup, recovery and patching.

Amazon RDS facilitates the deployment and maintenance of relational databases in the cloud. A cloud administrator uses Amazon RDS to set up, operate, manage and scale a relational instance of a cloud database. Amazon RDS is not itself a database; it is a service used to manage relational databases.

[__S3__](https://medium.com/analytics-vidhya/apache-spark-applications-with-amazon-emr-and-s3-services-using-jupyter-notebook-41968a1c2d7)
Amazon Simple Storage Service (Amazon S3) offers an object storage service for the internet which is designed to store data (up to 5 terabytes single file) from any kind of source with providing scalability, data availability, security, and performance.

## Preparing The Environment For The Migration <a name="preparingenvironment"></a>

## Creating S3 Bucket <a name="creatings3bucket"></a>

You can choose two different ways for the s3 bucket creation. It can be done either AWS Management Console or AWS Toolkit for Visual Studio Code.

### AWS Management Console

To create a S3 bucket on Console just look for s3 by typing it.

<img src="https://user-images.githubusercontent.com/69978184/144335790-b755cc37-3e31-4671-8617-c105bb6dd17b.png" width="800" height="400"/>

and then, just click on the orange button which description is "Create Bucket" in order to create bucket.

<img src="https://user-images.githubusercontent.com/69978184/144336501-3de8091d-e40e-4aaa-b7e2-c01f9573d508.png" width="800" height="400"/>

### AWS Toolkit for Visual Studio Code

To do this procedure is needed to install [VS Code](https://code.visualstudio.com/download) on your local Machine.

Once you've installed VSCode, some steps will be needed from now on. 

1) [Obtaining AWS Acess Keys](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/obtain-credentials.html)

2) How to [set up the AWS Credentials](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-credentials.html) on your Local Machine

3) [Connecting to AWS through the AWS Toolkit for Visual Studio Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/connect.html)

Once you've already followed the step-by-step guide above, if you acess your VSCode, you may see something like this:

<img src="https://user-images.githubusercontent.com/69978184/144343625-8553c492-8bcf-4a85-8b3e-8567acfcafe7.png" width="800" height="400"/>

It means that the connection between cloud and local machine has been completed Successfully.

Now, you can just click on the bucket object as shown on the image close s3

and then give the bucket name as you want
<img src="https://user-images.githubusercontent.com/69978184/144344451-90440fb8-4511-42b1-8255-f2eae7c4bce4.png" width="800" height="400"/>

The name I chose is dbdmsbucketforcsv as folows

<img src="https://user-images.githubusercontent.com/69978184/144345083-84ab18e9-a6b3-4a17-8937-f24edc7ed764.png" width="800" height="400"/>

As you can see, it was created normally.

## Creating Postgres RDS <a name="creatingRDS"></a>

To create a RDS Database click on Search Console and just look for RDS by typing it.

After that, click on Create Database button

<img src="https://user-images.githubusercontent.com/69978184/144347842-a60ac04c-8154-4826-a923-81abbe72cd6c.png" width="800" height="400"/>

In this step for the creation, be smart with the Postgres version, depending on the version you choose, the free tier is no longer available, that is why you have to be smart when creating this RDS on free tier mode, as follows below

<img src="https://user-images.githubusercontent.com/69978184/144349314-97929957-df27-44ba-9a9a-d3749ed6f879.png" width="800" height="400"/>

and then, keep going by filling those informations required fields. Just notice once you've selected free tier mode, many of these fields will be already filled for this mode mentioned.

<img src="https://user-images.githubusercontent.com/69978184/144349982-4b433035-75f7-4d0b-90d8-4b8d0f195b7e.png" width="800" height="400"/>

then create the database and you will get something like this

<img src="https://user-images.githubusercontent.com/69978184/144350657-3c26f156-3005-4e4b-9b84-4f2badfd68c0.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144351554-bd13a3b2-a563-48d9-a282-5f8dff88612f.png" width="800" height="400"/>

## Launching an EC2 instance <a name="launchingEC2"></a>

To create an EC2 instance on Console just look for EC2 by typing it.

After that, just follow this [step-by-step guide](https://www.guru99.com/creating-amazon-ec2-instance.html) to launch an EC2 instance

With its creation, this one will need to have the same VPC, consequently its security group needs to have the same ports 22 and 5432(belonging to Postgres) like on the RDS DB Instance created. To check this out better and how to do this staff correctly, just click on this [link](https://aws.amazon.com/premiumsupport/knowledge-center/connect-http-https-ec2/) 

<!--
## Connecting to EC2 instance <a name="connectingEC2"></a>

For now, let is connect to our EC2 instance recently create. For that, just go thorugh console and look for EC2 again and select the EC2 by checking the box, thus click on connect button to start the connection

<img src="https://user-images.githubusercontent.com/69978184/144669132-dacc2a51-12e3-4865-b4e1-08de2e7c7a6e.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144669273-1224e820-d0f8-48ad-b83a-0073247223ce.png" width="800" height="400"/>

Then a new windows will be opened via browser terminal

<img src="https://user-images.githubusercontent.com/69978184/144669686-085d81d6-b391-4732-8ee8-21dc9f0bce33.png" width="800" height="400"/>

there is also another way to open this terminal via Windows and this one will be applied than the first option for this case. 

If you wanna use the first option, feel free and choose that suits you best

Do you remember that file.pem generated after EC2 instance creation? Yes, you'll need to use that one as mentioned [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html) or [here](https://positive-stud.medium.com/how-to-use-putty-to-connect-to-your-amazon-ec2-instance-96468ce592c1), right now

After following the step-by-step above, the appearence of your screen will be like:

<img src="https://user-images.githubusercontent.com/69978184/144681178-793cc1b8-2c5e-49ef-ad37-2fd5e78d110e.png" width="800" height="400"/>
-->
## Connecting RDS locally with PGAdmin and Setting up Database <a name="connectingRDS"></a>

Start your pgAdmin and create your server in order to connect your RDS DB locally, but before that, go through your RDS DB Instance and check its endpoint

<img src="https://user-images.githubusercontent.com/69978184/144687682-63293438-9215-4246-a446-897802ec87d9.png" width="800" height="400"/>

once you obtained this endpoint, copy it and go to your pgAdmin Creation Server to paste it over there

<img src="https://user-images.githubusercontent.com/69978184/144687990-18799401-42eb-4006-a71f-3c00c1cefee9.png" width="500" height="400"/>

and it's done.

<img src="https://user-images.githubusercontent.com/69978184/144688268-8f4f6a83-c0c6-4b6b-b2f7-fb62dcaa53c9.png" width="800" height="400"/>

If you are facing some issues by setting it up, I recommend to read these three links, I'm sure that they may help you out :)

a) [First Link](https://stackoverflow.com/questions/61181015/postgres-pgadmin-unable-to-connect-to-server-timeout-expired)<br/>
b) [Second Link](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html#cfn-rds-dbinstance-publiclyaccessible)<br/>
c) [Third Link](https://stackoverflow.com/questions/61062027/aws-rds-to-pgadmin-error-saving-properties-unable-to-connect-to-server-timeout)<br/>

Before going to the next section, let's set up the database. For that, I shared a file whose name is SportsDB.sql in this repository to prepare our SportsDB Database, just download it in order to be created

<img src="https://user-images.githubusercontent.com/69978184/144688762-86fcaf16-b506-4b22-8c95-269153e7d170.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144688779-06e76bd6-a1f2-483c-8341-9a2b96305ab0.png" width="800" height="400"/>

Now on the query tool, we'll import the file mentioned previously here and run the query.

<img src="https://user-images.githubusercontent.com/69978184/144688944-4a53fca3-1af9-4581-a2ef-61091324f2b8.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144689464-d8134ed4-cd10-40c6-b9a4-de77c1de90d0.png" width="800" height="400"/>

# Migrating Data from RDS DB instance to S3 <a name="migratingdatafromRDS"></a>

## Creating IAM Role for S3 and RDS <a name="creatingIAMrole"></a>

Go to IAM section and click on create a role. After that, select the AWS Service called DMS and click on Next:Permissions

<img src="https://user-images.githubusercontent.com/69978184/144690764-d1e1a814-714c-456a-982f-27b1220bae84.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144690966-790bb8e7-a996-4d1b-bb88-86dbb6639de7.png" width="800" height="400"/>

In this step, look for S3 and select "AmazonS3FullAcess"

<img src="https://user-images.githubusercontent.com/69978184/144691199-cfe0b981-329e-4d7c-a1ca-f1f5f37ccb26.png" width="800" height="400"/>

To finish this step, give a role name to it and click on "Create Role"
<img src="https://user-images.githubusercontent.com/69978184/144691350-b68ef6a6-be0e-407a-a0fd-81990c86f5c4.png" width="800" height="400"/>

It's been created sucessfully

<img src="https://user-images.githubusercontent.com/69978184/144691492-d4c40138-6ba8-4903-9cf8-9164d24c3c6d.png" width="800" height="400"/>

but let's don't forget the RDS acess at all! So, let is do it :)

For that, click on Attach Policies button and look for RDS to select "AmazonRDSFullAccess"

<img src="https://user-images.githubusercontent.com/69978184/144691538-8851b245-e66d-44c9-8e65-e04edc3ce289.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144691752-b0e90a20-ba98-4ace-93d8-5eeb65b26b62.png" width="800" height="400"/>

## Creating a Replication Instance on DMS and Endpoints <a name="creatingreplication"></a>

Go to DMS section and click on replication instance and follow the steps below

<img src="https://user-images.githubusercontent.com/69978184/144692128-718fe58e-202d-496a-a8c0-a177f007f29b.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144692446-40208922-ed5d-46d3-b2d0-934ab351527a.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144692516-a53b96d0-b0ea-426f-8771-6ba3afd5b639.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144692549-ac0fe720-e3cd-43be-b463-28caa4d73321.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144692809-ff26710a-c6c1-45cb-942a-dc21aa55ea89.png" width="800" height="400"/>

and then create the end point for source and target

1) Source

<img src="https://user-images.githubusercontent.com/69978184/144692932-31a39ba8-d23a-4de0-9d46-e09034ef6781.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144693028-0248e0d8-5aac-4460-8eb1-a81cc14a0b53.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144693068-628bde29-867a-43e2-8454-b5a76f34ebf6.png" width="800" height="400"/>

Before carrying out, you can do a test in order to check wether the connection will work properly or not

<img src="https://user-images.githubusercontent.com/69978184/144693185-ba348757-d92b-4be5-9797-614f8e20f53f.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144693346-6dd99988-5e4b-492d-8605-eeec58ab06b1.png" width="800" height="400"/>

Goods news! Endpoint has been created successfully

<img src="https://user-images.githubusercontent.com/69978184/144693405-9704bf3d-3377-449f-95d0-e6235fbb7ce7.png" width="800" height="400"/>

2) Target

For target endpoint is requested that ARN given when the role was created previously on this tutorial. Don't forget to copy and paste it over here.

<img src="https://user-images.githubusercontent.com/69978184/144693656-a9e59ddc-72f4-495d-aa4f-c640298edbef.png" width="800" height="400"/>

let is do a new test for this target endpoint as well it has been done for the source endpoint

<img src="https://user-images.githubusercontent.com/69978184/144693700-23d9d387-6116-4115-aceb-e0a4ab258e53.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144693829-c1441965-e640-400e-95c0-c5f47bed58b6.png" width="800" height="400"/>

## Create and Run a Task For Migration <a name="formigration"></a>

<img src="https://user-images.githubusercontent.com/69978184/144694218-83f10d0f-b207-42ae-9ea8-3811a3929190.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144694249-2ca6cbd0-28e9-419a-8ff1-a162f70f4d18.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144694266-0f04abb6-2f86-4e09-9d99-88ccca559141.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144694303-eb4ebc73-8301-40a9-a614-327355c58445.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144694369-dfd0f2a8-7fdd-4e0f-b0ad-a7d0326e4ceb.png" width="800" height="400"/>

As soon as you run this job, you may notice that there will be an error and it has to do with [grants](https://forums.aws.amazon.com/thread.jspa?threadID=333935) as suggested on the available link. Thus, you may need to give grant to the user that is being used for this process, in my case is postgres, then I'll give grants to it.

<img src="https://user-images.githubusercontent.com/69978184/144697323-463a5d96-4d91-47d0-9ef5-857cee79629a.png" width="800" height="400"/>

<img src="https://user-images.githubusercontent.com/69978184/144700956-254a5090-5fb2-47ee-b0e8-d76ae83573f6.png" width="800" height="400"/>
