# Quotas and constraints for Amazon Aurora<a name="CHAP_Limits"></a>

Following, you can find a description of the resource quotas and naming constraints for Amazon Aurora\.

**Topics**
+ [Quotas in Amazon Aurora](#RDS_Limits.Limits)
+ [Naming constraints in Amazon Aurora](#RDS_Limits.Constraints)
+ [Amazon Aurora size limits](#RDS_Limits.FileSize.Aurora)

## Quotas in Amazon Aurora<a name="RDS_Limits.Limits"></a>

Each AWS account has quotas, for each AWS Region, on the number of Amazon Aurora resources that can be created\. After a quota for a resource has been reached, additional calls to create that resource fail with an exception\.

The following table lists the resources and their quotas per AWS Region\.


| Resource | Default quota | 
| --- | --- | 
| Authorizations per DB security group | 20 | 
| Concurrent DB snapshot export tasks | 5 | 
| Cross\-Region DB snapshot copy requests | 5 | 
| Custom endpoints per DB cluster | 5 | 
| Data API maximum result set size | 1 MB | 
| Data API requests per second | 1000 | 
| DB clusters | 40 | 
| DB cluster parameter groups | 50 | 
| DB instances | 40 | 
| DB subnet groups | 50 | 
| Event subscriptions | 20 | 
| AWS Identity and Access Management \(IAM\) roles per DB cluster | 5 | 
| IAM roles per DB instance | 5 | 
| Manual DB cluster snapshots | 100 | 
| Parameter groups | 50 | 
| Proxies | 20 | 
| Reserved DB instances | 40 | 
| Rules per virtual private cloud \(VPC\) security group | 50 inbound, 50 outbound | 
| Subnets per subnet group | 20 | 
| Tags per resource | 50 | 
| VPC security groups | 5 | 

**Note**  
By default, you can have up to a total of 40 DB instances\. RDS DB instances, Aurora DB instances, Amazon Neptune instances, and Amazon DocumentDB instances apply to this quota\.  
If your application requires more DB instances, you can request additional DB instances by opening the [Service Quotas console](https://console.aws.amazon.com/servicequotas/home?region=us-east-1#!/dashboard)\. In the navigation pane, choose **AWS services**\. Choose **Amazon Relational Database Service \(Amazon RDS\)**, choose a quota, and follow the directions to request a quota increase\. For more information, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-increase.html) in the *Service Quotas User Guide*\.  
Backups managed by AWS Backup are considered manual DB cluster snapshots, but don't count toward the manual cluster snapshot quota\. For information about AWS Backup, see the [https://docs.aws.amazon.com/aws-backup/latest/devguide](https://docs.aws.amazon.com/aws-backup/latest/devguide)\.

## Naming constraints in Amazon Aurora<a name="RDS_Limits.Constraints"></a>

The following table describes naming constraints in Amazon Aurora\. 


****  

| Resource or item | Constraints | 
| --- | --- | 
| DB cluster identifier |  Identifiers have these naming constraints: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html)  | 
|  Initial database name  |  Database name constraints differ between Aurora MySQL and PostgreSQL\. For more information, see the available settings when creating each DB cluster\.  | 
|  Master user name  |  Master user name constraints differ for each database engine\. For more information, see the available settings when creating each DB cluster\.  | 
|  Master password  |  The password for the database master user can include any printable ASCII character except `/`, `"`, `@`, or a space\. Master password length constraints differ for each database engine\. For more information, see the available settings when creating each DB cluster\.  | 
| DB parameter group name |  These names have these constraints: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html)  | 
|  DB subnet group name  |  These names have these constraints: [\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Limits.html)  | 

## Amazon Aurora size limits<a name="RDS_Limits.FileSize.Aurora"></a>

**Storage size limits**  
An Aurora cluster volume can grow to a maximum size of 128 tebibytes \(TiB\) for the following engine versions:  
+ Aurora MySQL engine versions 2\.09 compatible with MySQL 5\.70 and 1\.23 compatible with MySQL 5\.6\.
+ Aurora PostgreSQL engine versions 12\.4 and higher, 11\.7 and higher, 10\.12 and higher, and 9\.6\.17 and higher\.
For lower engine versions, the maximum size of an Aurora cluster volume is 64 TiB\. For more information, see [How Aurora storage automatically resizes](Aurora.Overview.StorageReliability.md#aurora-storage-growth)\.

**SQL table size limits**  
For Aurora MySQL, the maximum table size is 64 tebibytes \(TiB\)\. For an Aurora PostgreSQL DB cluster, the maximum table size is 32 tebibytes \(TiB\)\. We recommend that you follow table design best practices, such as partitioning of large tables\.

