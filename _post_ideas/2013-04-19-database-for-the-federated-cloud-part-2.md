---
layout: post
title: "Database for the Federated Cloud (Part Two)"
tags : [cloud, database, federated cloud ]
tweet: Take a look at this post on database for the federated cloud
hashtags: #cloud #database
bitmark: http://bit.ly/1195OaK
---
{% include JB/setup %}



Introduction
============
This is part two of an article on database for the federated cloud.
[Part one](/2013/04/16/database-for-the-federated-cloud-part-1) presented our model.  Here we will look at
existing database providers and see how they can fit it.

Technologies
============

Distributed Key Value
---------------------

[InfiniSpan](http://www.jboss.org/infinispan/)
[Voldemort](http://project-voldemort.com/)
[Riak](http://wiki.basho.com/)
[Hibari](https://github.com/hibari/hibari)

[Dynamo](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)


MySQL Clustering
================

http://www.codership.com/
http://www.severalnines.com/
http://www.mysql.com/products/cluster/
https://dev.mysql.com/tech-resources/articles/cluster-7.3-dmr.html


Distributed File System
=======================
GFS
HDFS


Batch Data Analysis
===================
Map Reduce
Hadoop


Real time Data analysis
=======================
Dremel
Apache Drill
Cloudera impala
hortonworks tez/stinger


Graph Databases
===============
Pregel
Apache Giraph
Titan
MS Trinity
Neo4j

Large Row
========
BigTable
[Accumulo](http://accumulo.apache.org/) on top of Hadoop
[Hypertable](http://www.hypertable.com/)
[HBase](http://hbase.apache.org/)
[Cassandra](http://cassandra.apache.org/) + Priam


Document Database
=================
[mongoDB](http://www.mongodb.org/)
[BigCouch](https://github.com/cloudant/bigcouch)
[CouchDB](http://couchdb.apache.org/)


HDFS replacements for Hadoop
============================
from http://gigaom.com/2012/07/11/because-hadoop-isnt-perfect-8-ways-to-replace-hdfs/
Cassandra (DataStax)
[Ceph](http://www.inktank.com/)
[Lustre](lustre.org)



EU Grid Projects
===========
http://www.egi.eu/


cluster controller
==================
[Mesos](http://incubator.apache.org/mesos/)



Apache Hive, Apache Pig and Cascading



