---
layout: post
title: Practice on Cassandra
date: 2019-03-15
tag: Distributed System
---

Cassandra is one of the most famous distributed databases. It is good at scalability and availability. Most importantly, it is estimated that Cassandra is 100X faster in reading than a commonly used mySql DB. The job in our project is read-heavy and Cassandra is a super fit for this. In our cluster, we stored TBs of data in Cassandra. The initial loading of data brought a lot of trouble to us. But people learn from failures. The following is what I learnt from my practice.


## How data gets written into Cassandra

Cassandra processes data at several stages on the write path, starting with the immediate logging of a write and ending in with a write of data to disk:

* Logging data in the commit log
* Writing data to the memtable
* Flushing data from the memtable
* Storing data on disk in SSTables

The write flow is show in the following digram.

![Cassandra write flow](/assets/images/cassandra_write_flow.png)

 Cassandra stores the data memtable, as well as appends writes to the commit log on disk. The commit log receives every write made to a Cassandra node, and these durable writes survive permanently even if power fails on a node.

 One of the problems we encountered when loading data into Cassandra is that the commit lob is overwhelmed. The default size of the commit log is 32Mb and only half of it could be used to store the write. If we have a record with a size over 16Mb, it's going to trigger errors when writing. We solve this problem by increasing the size of commit log.

## How data gets updated and deleted in Cassandra

Cassandra is using a different mechanism when comes to update/delete data, compared to mySql. I put update and delete in the same place here because there is in fact no "delete" in Cassandra. It is actually "update".

During a write, Cassandra adds each new row to the database without checking on whether a duplicate record exists. This policy makes it possible that many versions of the same row may exist in the database.

Periodically, the rows stored in memory are streamed to disk into SSTables. At certain intervals, Cassandra **compacts** smaller SSTables into larger SSTables. If Cassandra encounters two or more versions of the same row during this process, Cassandra only writes the most recent version to the new SSTable. After compaction, Cassandra drops the original SSTables, deleting the outdated rows. SSTable is immutable and Cassandra never modifies an existing SSTable.

![campaction](/assets/images/cassandra_compaction.png)

When a client requests data with a particular primary key, Cassandra retrieves many versions of the row from one or more replicas. The version with the most recent timestamp is the only one returned to the client ("last-write-wins").

As for delete, the data to be deleted will be added with a deletion marker called a tombstone. The tombstones go through Cassandra's write path, and are written to SSTables on one or more nodes. The key difference feature of a tombstone: it has a built-in expiration date/time. At the end of its expiration period (for details see below) the tombstone is deleted as part of Cassandra's normal compaction process.



## Primary Key, Composite Key, Partition Key and Clustering Key

Gee! So many keys! What are they? I know there is Primary Key in mySql. Is it necessary to have so many different keys in Cassandra?  

Let's see what they are and their functionalities.

* Primary Key:

    A primary key identifies the location and order of stored data. It is more complicated than the concept of "primary key" in sql. Cassandra is a partition row store. The primary key of a record determines which partition the record is going to be stored (which node it is going to be stored) and how it is going to be store. The definition of a table's primary key is critical in Cassandra. Carefully model how data in a table will be inserted and retrieved before choosing which columns to define in the primary key.
    When we want to create a primary key, we can use a command like this:
    ```sql
    create table people (
        ssn text PRIMARY KEY,
        name text      
    );
    ```

* Composite KEY

    A primary key could also be **COMPOSITE** or **COMPOUND**, generated from multiple columns.

    ```sql
    create table people (
        ssn text,
        dob text,
        gender text,
        name text,
        PRIMARY KEY(ssn, gender, dob)   
    );
    ```
* Partition Key and Clustering Key

    We already know that the Primary Key determines the partition. In a situation of **COMPOSITE Primary Key**, the "first part" of the key is called **PARTITION KEY** and the left part of the key is the **CLUSTERING KEY**. In the above example, ssn would be the Partition Key. gender and dob would be the Clustering Key. Partition Key determine the partition / which node to store the record. And Clustering Key determines the data sorting within the partition(Yes, SSTable means "sorted string table". It requires content been sorted to increase efficiency).

    There is another way to write a compound Primary Key. Look at the following example:

    ```sql
    CREATE TABLE phone (
        phone_nb text,
        phone_area_nb text,
        owner text
        PRIMARY KEY ((phone_nb, phone_area_nb), owner)
    );
    ```

    Here the partition key is (phone_nb, phone_are_nb), a combination.

* How where query works in CQL

    CQL is the language used for Cassandra. It's very similar to sql but it becomes tricky when it comes to "where" command. The main restriction is that you could not use "where" clause on non-primary keys. The "general" rule to make query is that you have to pass all partition key columns. You can choose to pass clustering keys too but it must be following the same order as you create the table. You could not skip one clustering key and use the one following it. We can look the following example:

    ```sql
    PRIMARY KEY((col1, col2), col3, col4))
    ```

    so the valid queries are: (without secondary indexing)
    >col1 and col2
    >
    >col1 and col2 and col3
    >
    >col1 and col2 and col3 and col 4

    Invalid:
    >col1 and col2 and col4
    >
    >anything that does not contain both col1 and col2

    Cassadra does not work the same way mySql works. Keep this in mind, if you find sth working in sql but not in cql, don't be surprised, google it :>

## The time out error we encountered when reading

After loading data into Cassandra, we want to count the records in one of the tables to make sure all data has been loaded. We used Spark to read the data out as a Dataframe and do a count on it. The problem we met is that when reading the data, we got this error:

```
py4j.protocol.Py4JJavaError: An error occurred while calling o125.count.
: org.apache.spark.SparkException: Job aborted due to stage failure: Task 215 in stage 23.0 failed 10 times,
most recent failure: Lost task 215.9 in stage 23.0 (TID 28322, 10.249.195.40, executor 39):
java.io.IOException: Exception during execution of SELECT count(*) FROM "us"."buid_eid_0dot7" WHERE
token("buid") > ? AND token("buid") <= ?   ALLOW FILTERING: Cassandra timeout during read query at
consistency LOCAL_ONE (1 responses were required but only 0 replica responded)
```

Don't know exactly why this happened. My guess is that the table contains too many columns that Spark could not read the whole table before timing out. I later use another method to get the count, which is to "copy" the whole table to NULL. After copying, the log will also tell me how many rows have been copied.



----------------------
reference

[https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlHowDataWritten.html](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlHowDataWritten.html)

[https://stackoverflow.com/questions/24949676/difference-between-partition-key-composite-key-and-clustering-key-in-cassandra](https://stackoverflow.com/questions/24949676/difference-between-partition-key-composite-key-and-clustering-key-in-cassandra)

[https://www.datastax.com/dev/blog/a-deep-look-to-the-cql-where-clause](https://www.datastax.com/dev/blog/a-deep-look-to-the-cql-where-clause)


