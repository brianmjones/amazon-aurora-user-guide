# Best practices with Amazon Aurora MySQL<a name="AuroraMySQL.BestPractices"></a>

This topic includes information on best practices and options for using or migrating data to an Amazon Aurora MySQL DB cluster\.

**Topics**
+ [Determining which DB instance you are connected to](#AuroraMySQL.BestPractices.DeterminePrimaryInstanceConnection)
+ [Using T2 and T3 instances](#AuroraMySQL.BestPractices.T2Medium)
+ [Invoking an AWS Lambda function](#AuroraMySQL.BestPractices.Lambda)
+ [Working with asynchronous key prefetch in Amazon Aurora](#Aurora.BestPractices.AKP)
+ [Working with multi\-threaded replication in Amazon Aurora MySQL](#AuroraMySQL.BestPractices.MTReplica)
+ [Using Amazon Aurora to scale reads for your MySQL database](#AuroraMySQL.BestPractices.ReadScaling)
+ [Using Amazon Aurora for Disaster Recovery with your MySQL databases](#AuroraMySQL.BestPractices.DisasterRecovery)
+ [Migrating from MySQL to Amazon Aurora MySQL with reduced downtime](#AuroraMySQL.BestPractices.Migrating)
+ [Using XA transactions with Amazon Aurora MySQL](#AuroraMySQL.BestPractices.XA)
+ [Working with hash joins in Aurora MySQL](#Aurora.BestPractices.HashJoin)
+ [Working with foreign keys in Aurora MySQL](#Aurora.BestPractices.ForeignKeys)
+ [Related topics](#AuroraMySQL.BestPractices.RelatedTopics)

## Determining which DB instance you are connected to<a name="AuroraMySQL.BestPractices.DeterminePrimaryInstanceConnection"></a>

To determine which DB instance in an Aurora MySQL DB cluster a connection is connected to, check the `innodb_read_only` global variable, as shown in the following example\.

```
SHOW GLOBAL VARIABLES LIKE 'innodb_read_only'; 
```

The `innodb_read_only` variable is set to `ON` if you are connected to an Aurora Replica and `OFF` if you are connected to the primary instance\.

This approach can be helpful if you want to add logic to your application code to balance the workload or to ensure that a write operation is using the correct connection\. This technique only applies to Aurora clusters using single\-master replication\. For multi\-master clusters, all the DB instances have the setting `innodb_read_only=OFF`\.

## Using T2 and T3 instances<a name="AuroraMySQL.BestPractices.T2Medium"></a>

Amazon Aurora MySQL instances that use the `db.t2.small` or `db.t2.medium` DB instance classes are best suited for applications that do not support a high workload for an extended amount of time\. T2 instances are designed to provide moderate baseline performance and the capability to burst to significantly higher performance as required by your workload\. They are intended for workloads that don't use the full CPU often or consistently, but occasionally need to burst\. We recommend only using the `db.t2.small` and `db.t2.medium` DB instance classes for development and test servers, or other non\-production servers\. For more details on T2 instances, see [T2 instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html)\.

If your Aurora cluster is larger than 40 TB, don't use db\.t2 or db\.t3 instance classes\.

Don't enable the MySQL Performance Schema on Amazon Aurora MySQL T2 instances\. If the Performance Schema is enabled, the T2 instance might run out of memory\.

When you use a T2 instance as a DB instance in an Aurora MySQL DB cluster, we recommend the following:
+ If you use a T2 instance as a DB instance class in your DB cluster, then we recommend that all instances in your DB cluster use the same DB instance class\. For example, if you use `db.t2.medium` for your primary instance, then we recommend that you use `db.t2.medium` for your Aurora Replicas as well\.
+ Monitor your CPU Credit Balance \(`CPUCreditBalance`\) to ensure that it is at a sustainable level\. That is, CPU credits are being accumulated at the same rate as they are being used\.

  When you have exhausted the CPU credits for an instance, you see an immediate drop in the available CPU and an increase in the read and write latency for the instance\. This situation results in a severe decrease in the overall performance of the instance\.

  If your CPU credit balance is not at a sustainable level, then we recommend that you modify your DB instance to use a one of the supported R3 DB instance classes \(scale compute\)\.

  For more information on monitoring metrics, see [Monitoring Amazon Aurora metrics with Amazon CloudWatch](Aurora.Monitoring.md)\.
+ For your Aurora MySQL DB clusters using single\-master replication, monitor the replica lag \(`AuroraReplicaLag`\) between the primary instance and the Aurora Replicas\.

  If an Aurora Replica runs out of CPU credits before the primary instance, the lag behind the primary instance results in the Aurora Replica frequently restarting\. This result is common when an application has a heavy load of read operations distributed among Aurora Replicas in an Aurora MySQL DB cluster, at the same time that the primary instance has a minimal load of write operations\.

  If you see a sustained increase in replica lag, make sure that your CPU credit balance for the Aurora Replicas in your DB cluster is not being exhausted\.

  If your CPU credit balance is not at a sustainable level, then we recommend that you modify your DB instance to use one of the supported R3 DB instance classes \(scale compute\)\.
+ Keep the number of inserts per transaction below 1 million for DB clusters that have binary logging enabled\.

  If the DB cluster parameter group for your DB cluster has the `binlog_format` parameter set to a value other than `OFF`, then your DB cluster might experience out\-of\-memory conditions if the DB cluster receives transactions that contain over 1 million rows to insert\. You can monitor the freeable memory \(`FreeableMemory`\) metric to determine if your DB cluster is running out of available memory\. You then check the write operations \(`VolumeWriteIOPS`\) metric to see if a writer instance is receiving a heavy load of write operations\. If this is the case, then we recommend that you update your application to limit the number of inserts in a transaction to less than 1 million\. Alternatively, you can modify your instance to use one of the supported R3 DB instance classes \(scale compute\)\.

## Invoking an AWS Lambda function<a name="AuroraMySQL.BestPractices.Lambda"></a>

If you are using Amazon Aurora version 1\.16 or later, we recommend using the native functions `lambda_sync` and `lambda_async` to invoke Lambda functions\.

If you are using the deprecated `mysql.lambda_async` procedure, we recommend that you wrap calls to the `mysql.lambda_async` procedure in a stored procedure\. You can call this stored procedure from different sources, such as triggers or client code\. This approach can help to avoid impedance mismatch issues and make it easier for your database programmers to invoke Lambda functions\.

For more information on invoking Lambda functions from Amazon Aurora, see [Invoking a Lambda function from an Amazon Aurora MySQL DB cluster](AuroraMySQL.Integrating.Lambda.md)\.

## Working with asynchronous key prefetch in Amazon Aurora<a name="Aurora.BestPractices.AKP"></a>

**Note**  
The asynchronous key prefetch \(AKP\) feature is available for Amazon Aurora MySQL version 1\.15 and later\. For more information about Aurora MySQL versions, see [Database engine updates for Amazon Aurora MySQL](AuroraMySQL.Updates.md)\.

Amazon Aurora can use AKP to improve the performance of queries that join tables across indexes\. This feature improves performance by anticipating the rows needed to run queries in which a JOIN query requires use of the Batched Key Access \(BKA\) Join algorithm and Multi\-Range Read \(MRR\) optimization features\. For more information about BKA and MRR, see [Block nested\-loop and batched key access joins](https://dev.mysql.com/doc/refman/5.6/en/bnl-bka-optimization.html) and [Multi\-range read optimization](https://dev.mysql.com/doc/refman/5.6/en/mrr-optimization.html) in the MySQL documentation\.

To take advantage of the AKP feature, a query must use both BKA and MRR\. Typically, such a query occurs when the JOIN clause of a query uses a secondary index, but also needs some columns from the primary index\. For example, you can use AKP when a JOIN clause represents an equijoin on index values between a small outer and large inner table, and the index is highly selective on the larger table\. AKP works in concert with BKA and MRR to perform a secondary to primary index lookup during the evaluation of the JOIN clause\. AKP identifies the rows required to run the query during the evaluation of the JOIN clause\. It then uses a background thread to asynchronously load the pages containing those rows into memory before running the query\.

### Enabling asynchronous key prefetch<a name="Aurora.BestPractices.AKP.Enabling"></a>

You can enable the AKP feature by setting `aurora_use_key_prefetch`, a MySQL server variable, to `on`\. By default, this value is set to `on`\. However, AKP cannot be enabled until you also enable the BKA Join algorithm and disable cost\-based MRR functionality\. To do so, you must set the following values for `optimizer_switch`, a MySQL server variable:
+ Set `batched_key_access` to `on`\. This value controls the use of the BKA Join algorithm\. By default, this value is set to `off`\.
+ Set `mrr_cost_based` to `off`\. This value controls the use of cost\-based MRR functionality\. By default, this value is set to `on`\.

Currently, you can set these values only at the session level\. The following example illustrates how to set these values to enable AKP for the current session by executing SET statements\.

```
mysql> set @@session.aurora_use_key_prefetch=on;
mysql> set @@session.optimizer_switch='batched_key_access=on,mrr_cost_based=off';
```

Similarly, you can use SET statements to disable AKP and the BKA Join algorithm and re\-enable cost\-based MRR functionality for the current session, as shown in the following example\.

```
mysql> set @@session.aurora_use_key_prefetch=off;
mysql> set @@session.optimizer_switch='batched_key_access=off,mrr_cost_based=on';
```

For more information about the **batched\_key\_access** and **mrr\_cost\_based** optimizer switches, see [Switchable optimizations](https://dev.mysql.com/doc/refman/5.6/en/switchable-optimizations.html) in the MySQL documentation\.

### Optimizing queries for asynchronous key prefetch<a name="Aurora.BestPractices.AKP.Optimizing"></a>

You can confirm whether a query can take advantage of the AKP feature\. To do so, use the EXPLAIN statement with the EXTENDED keyword to profile the query before running it\. The *EXPLAIN statement* provides information about the execution plan to use for a specified query\.

In the output for the EXPLAIN statement, the `Extra` column describes additional information included with the execution plan\. If the AKP feature applies to a table used in the query, this column includes one of the following values:
+ `Using Key Prefetching`
+ `Using join buffer (Batched Key Access with Key Prefetching)`

The following example shows use of EXPLAIN with EXTENDED to view the execution plan for a query that can take advantage of AKP\.

```
mysql> explain extended select sql_no_cache
    ->     ps_partkey,
    ->     sum(ps_supplycost * ps_availqty) as value
    -> from
    ->     partsupp,
    ->     supplier,
    ->     nation
    -> where
    ->     ps_suppkey = s_suppkey
    ->     and s_nationkey = n_nationkey
    ->     and n_name = 'ETHIOPIA'
    -> group by
    ->     ps_partkey having
    ->         sum(ps_supplycost * ps_availqty) > (
    ->             select
    ->                 sum(ps_supplycost * ps_availqty) * 0.0000003333
    ->             from
    ->                 partsupp,
    ->                 supplier,
    ->                 nation
    ->             where
    ->                 ps_suppkey = s_suppkey
    ->                 and s_nationkey = n_nationkey
    ->                 and n_name = 'ETHIOPIA'
    ->         )
    -> order by
    ->     value desc;
+----+-------------+----------+------+-----------------------+---------------+---------+----------------------------------+------+----------+-------------------------------------------------------------+
| id | select_type | table    | type | possible_keys         | key           | key_len | ref                              | rows | filtered | Extra                                                       |
+----+-------------+----------+------+-----------------------+---------------+---------+----------------------------------+------+----------+-------------------------------------------------------------+
|  1 | PRIMARY     | nation   | ALL  | PRIMARY               | NULL          | NULL    | NULL                             |   25 |   100.00 | Using where; Using temporary; Using filesort                |
|  1 | PRIMARY     | supplier | ref  | PRIMARY,i_s_nationkey | i_s_nationkey | 5       | dbt3_scale_10.nation.n_nationkey | 2057 |   100.00 | Using index                                                 |
|  1 | PRIMARY     | partsupp | ref  | i_ps_suppkey          | i_ps_suppkey  | 4       | dbt3_scale_10.supplier.s_suppkey |   42 |   100.00 | Using join buffer (Batched Key Access with Key Prefetching) |
|  2 | SUBQUERY    | nation   | ALL  | PRIMARY               | NULL          | NULL    | NULL                             |   25 |   100.00 | Using where                                                 |
|  2 | SUBQUERY    | supplier | ref  | PRIMARY,i_s_nationkey | i_s_nationkey | 5       | dbt3_scale_10.nation.n_nationkey | 2057 |   100.00 | Using index                                                 |
|  2 | SUBQUERY    | partsupp | ref  | i_ps_suppkey          | i_ps_suppkey  | 4       | dbt3_scale_10.supplier.s_suppkey |   42 |   100.00 | Using join buffer (Batched Key Access with Key Prefetching) |
+----+-------------+----------+------+-----------------------+---------------+---------+----------------------------------+------+----------+-------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
```

For more information about the extended `EXPLAIN` output format, see [Extended EXPLAIN output format](https://dev.mysql.com/doc/refman/5.6/en/explain-extended.html) in the MySQL product documentation\.

## Working with multi\-threaded replication in Amazon Aurora MySQL<a name="AuroraMySQL.BestPractices.MTReplica"></a>

By default, Aurora uses single\-threaded replication when an Aurora MySQL DB cluster is used as a read replica for binary log replication\. While Amazon Aurora doesn't prohibit multithreaded replication, Aurora MySQL has inherited several issues regarding multithreaded replication from MySQL\. We recommend that you do not use multithreaded replication in production\. If you do use multithreaded replication, we recommend that you test any use thoroughly\.

For more information about using replication in Amazon Aurora, see [Replication with Amazon Aurora](Aurora.Replication.md)\.

## Using Amazon Aurora to scale reads for your MySQL database<a name="AuroraMySQL.BestPractices.ReadScaling"></a>

You can use Amazon Aurora with your MySQL DB instance to take advantage of the read scaling capabilities of Amazon Aurora and expand the read workload for your MySQL DB instance\. To use Aurora to read scale your MySQL DB instance, create an Aurora MySQL DB cluster and make it a read replica of your MySQL DB instance\. Then connect to the Aurora MySQL cluster to process the read queries\. The source database can be an RDS for MySQL DB instance, or a MySQL database running external to Amazon RDS\. For more information, see [Using Amazon Aurora to scale reads for your MySQL database](AuroraMySQL.Replication.MySQL.md#AuroraMySQL.Replication.ReadScaling)\.

## Using Amazon Aurora for Disaster Recovery with your MySQL databases<a name="AuroraMySQL.BestPractices.DisasterRecovery"></a>

You can use Amazon Aurora with your MySQL DB instance to create an offsite backup for disaster recovery\. To use Aurora for disaster recovery of your MySQL DB instance, create an Amazon Aurora DB cluster and make it a read replica of your MySQL DB instance\. This applies to an RDS for MySQL DB instance, or a MySQL database running external to Amazon RDS\.

**Important**  
When you set up replication between a MySQL DB instance and an Amazon Aurora MySQL DB cluster, you should monitor the replication to ensure that it remains healthy and repair it if necessary\.

For instructions on how to create an Amazon Aurora MySQL DB cluster and make it a read replica of your MySQL DB instance, follow the procedure in [Using Amazon Aurora to scale reads for your MySQL database](#AuroraMySQL.BestPractices.ReadScaling)\.

## Migrating from MySQL to Amazon Aurora MySQL with reduced downtime<a name="AuroraMySQL.BestPractices.Migrating"></a>

When importing data from a MySQL database that supports a live application to an Amazon Aurora MySQL DB cluster, you might want to reduce the time that service is interrupted while you migrate\. To do so, you can use the procedure documented in [ Importing data to a MySQL or MariaDB DB instance with reduced downtime](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MySQL.Procedural.Importing.NonRDSRepl.html) in the *Amazon Relational Database Service User Guide*\. This procedure can especially help if you are working with a very large database\. You can use the procedure to reduce the cost of the import by minimizing the amount of data that is passed across the network to AWS\.

The procedure lists steps to transfer a copy of your database data to an Amazon EC2 instance and import the data into a new RDS for MySQL DB instance\. Because Amazon Aurora is compatible with MySQL, you can instead use an Amazon Aurora DB cluster for the target Amazon RDS MySQL DB instance\.

## Using XA transactions with Amazon Aurora MySQL<a name="AuroraMySQL.BestPractices.XA"></a>

We recommend that you don't use eXtended Architecture \(XA\) transactions with Aurora MySQL, because they can cause long recovery times if the XA was in the `PREPARED` state\. If you must use XA transactions with Aurora MySQL, follow these best practices:
+ Don't leave an XA transaction open in the `PREPARED` state\.
+ Keep XA transactions as small as possible\.

For more information about using XA transactions with MySQL, see [XA transactions](https://dev.mysql.com/doc/refman/5.6/en/xa.html) in the MySQL documentation\.

## Working with hash joins in Aurora MySQL<a name="Aurora.BestPractices.HashJoin"></a>

When you need to join a large amount of data by using an equijoin, a hash join can improve query performance\. You can enable hash joins for Aurora MySQL\.

A hash join column can be any complex expression\. In a hash join column, you can compare across data types in the following ways:
+ You can compare anything across the category of precise numeric data types, such as `int`, `bigint`, `numeric`, and `bit`\.
+ You can compare anything across the category of approximate numeric data types, such as `float` and `double`\.
+ You can compare items across string types if the string types have the same character set and collation\.
+ You can compare items with date and timestamp data types if the types are the same\.

**Note**  
Data types in different categories cannot compare\.

The following restrictions apply to hash joins for Aurora MySQL:
+ Left\-right outer joins are not supported\.
+ Semijoins such as subqueries are not supported, unless the subqueries are materialized first\.
+ Multiple\-table updates or deletes are not supported\.
**Note**  
Single\-table updates or deletes are supported\.
+ BLOB and spatial data type columns cannot be join columns in a hash join\.

### Enabling hash joins<a name="Aurora.BestPractices.HashJoin.Enabling"></a>

To enable hash joins, set the MySQL server variable `optimizer_switch` to `hash_join=on`\. The `optimizer_switch` parameter is set to `hash_join=off` by default\. The following example illustrates how to enable hash joins\.

```
mysql> SET optimizer_switch='hash_join=on';
```

With this setting, the optimizer chooses to use a hash join based on cost, query characteristics, and resource availability\. If the cost estimation is incorrect, you can force the optimizer to choose a hash join\. You do so by setting `hash_join_cost_based`, a MySQL server variable, to `off`\. The following example illustrates how to force the optimizer to choose a hash join\.

```
mysql> SET optimizer_switch='hash_join_cost_based=off';
```

**Note**  
 Prior to Aurora MySQL version 1\.22, the way to enable hash joins in Aurora MySQL version 1 is by enabling the `aurora_lab_mode` session\-level setting\. In those Aurora MySQL versions, the `optimizer_switch` setting for hash joins is enabled by default and you only need to enable `aurora_lab_mode`\.   
 For Aurora MySQL version 2, hash join support is available in version 2\.06 and higher\. In Aurora MySQL version 2, the hash join feature is always controlled by the `optimizer_switch` value\. 

### Optimizing queries for hash joins<a name="Aurora.BestPractices.HashJoin.Optimizing"></a>

To find out whether a query can take advantage of a hash join, use the `EXPLAIN` statement to profile the query first\. The `EXPLAIN` statement provides information about the execution plan to use for a specified query\.

In the output for the `EXPLAIN` statement, the `Extra` column describes additional information included with the execution plan\. If a hash join applies to the tables used in the query, this column includes values similar to the following:
+ `Using where; Using join buffer (Hash Join Outer table table1_name)`
+ `Using where; Using join buffer (Hash Join Inner table table2_name)`

The following example shows the use of EXPLAIN to view the execution plan for a hash join query\.

```
mysql> explain SELECT sql_no_cache * FROM hj_small, hj_big, hj_big2
    ->     WHERE hj_small.col1 = hj_big.col1 and hj_big.col1=hj_big2.col1 ORDER BY 1;
+----+-------------+----------+------+---------------+------+---------+------+------+----------------------------------------------------------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra                                                          |
+----+-------------+----------+------+---------------+------+---------+------+------+----------------------------------------------------------------+
|  1 | SIMPLE      | hj_small | ALL  | NULL          | NULL | NULL    | NULL |    6 | Using temporary; Using filesort                                |
|  1 | SIMPLE      | hj_big   | ALL  | NULL          | NULL | NULL    | NULL |   10 | Using where; Using join buffer (Hash Join Outer table hj_big)  |
|  1 | SIMPLE      | hj_big2  | ALL  | NULL          | NULL | NULL    | NULL |   15 | Using where; Using join buffer (Hash Join Inner table hj_big2) |
+----+-------------+----------+------+---------------+------+---------+------+------+----------------------------------------------------------------+
3 rows in set (0.04 sec)
```

In the output, the `Hash Join Inner table` is the table used to build hash table, and the `Hash Join Outer table` is the table that is used to probe the hash table\.

For more information about the extended `EXPLAIN` output format, see [Extended EXPLAIN output format](https://dev.mysql.com/doc/refman/5.6/en/explain-extended.html) in the MySQL product documentation\.

 In Aurora MySQL 2\.08 and higher, you can use SQL hints to influence whether a query uses hash join or not, and which tables to use for the build and probe sides of the join\. For details, see [Aurora MySQL hints](AuroraMySQL.Reference.md#AuroraMySQL.Reference.Hints)\. 

## Working with foreign keys in Aurora MySQL<a name="Aurora.BestPractices.ForeignKeys"></a>

We strongly recommend that you don't run any data definition language \(DDL\) statements when the `foreign_key_checks` variable is set to `0` \(off\)\.

If you need to insert or update rows that require a transient violation of foreign keys, follow these steps:

1. Set `foreign_key_checks` to `0`\.

1. Make your data manipulation language \(DML\) changes\.

1. Make sure that your completed changes don't violate any foreign key constraints\.

1. Set `foreign_key_checks` to `1` \(on\)\.

In addition, follow these other best practices for foreign key constraints:
+ Make sure that your client applications don't set the `foreign_key_checks` variable to `0` as a part of the `init_connect` variable\.
+ If a restore from a logical backup such as `mysqldump` fails or is incomplete, make sure that `foreign_key_checks` is set to `1` before starting any other operations in the same session\. A logical backup sets `foreign_key_checks` to `0` when it starts\.

## Related topics<a name="AuroraMySQL.BestPractices.RelatedTopics"></a>
+ [Managing an Amazon Aurora DB cluster](CHAP_Aurora.md)