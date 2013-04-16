---
layout: post
title: "Database for the Federated Cloud (Part One)"
tags : [cloud, database, federated cloud ]
tweet:
hashtags: #cloud #database
bitmark:
---
{% include JB/setup %}

Introduction
============
This article describes how federated database will be key to the federated cloud.

In part one, we will describe how entreprise architecture has changed with the arrival of
the cloud and how we think it will evolve in the future.  In part two, we will give a list of database technologies that
will support a federated cloud model.

Federated Cloud
===============
A cloud infrastructure is federated when it combines the resources of different cloud providers.
Federating your cloud solution has the following benefits:
* no vendor locking,
* higher availability if your application is designed properly.

The related costs are:
* higher complexity due to different infrastructure capabilities and API,
* multiple provider relationships and billing,
* higher networking cost.

While using a federated cloud, we still want centralized
* management interface,
* monitoring,
* billing

In addition, we still need to share our data between different instances.

Architecture
============

Before the Cloud
--------------------------
Before the cloud and still for the majority of companies, IT infrastructure is characterised by some computation,
network and storage resources manually provisioned in one or more dedicated datacentres after a lengthy procurement
process.

With the help of some naming services (DNS) and maybe some content delivery networks (CDN), external users will access
one or more services through different access points depending on criteria like
user experience (the closest the better), local laws (payment card regulations) or availability.
Internal users will access the services from an internal private network.

The database will often be centralized with a mix of highly transactional, partitioned and read only content.

While this model has demonstrated its value,
* it has a considerable setup cost and time and
* it is not elastic, i.e. it does not scale out or scale down on demand,

Elasticity is necessary to reduce risk and encourage business agility.

Below is a very simplified representation of a company IT infrastructure before the cloud:

<img class="diagram" alt='Long Feedback Loop' src='/assets/drawings/federated-db/before-the-cloud.png'/>

With the Cloud
-----------------------
With the advance of the cloud, infrastructure becomes a commodity.  It can be deployed, scaled and
decommissioned on demand while only paying for the actual usage.

Companies can offload their rigid private datacentres and gain more agility.  Database can be partially moved to the
cloud depending on the usage pattern:
* read only and partitioned databases will be prime candidate for migration,
* highly transactional databases will remain centralized, either in private datacentres or on a limited number of
cloud infrastructures, resulting in uneven performance depending the origin of the request.

In this model, all users, internal and external, access the service through the Internet.

To avoid vendor locking and improve their resilience, companies can use a federated cloud.

In our simplified diagram below, the application front-end as well as the read-only and partitioned databases have been
moved to the cloud.  The transactional data have remained on the private infrastructure but could have been moved to
one of the cloud provider:

<img class="diagram" alt='Long Feedback Loop' src='/assets/drawings/federated-db/with-the-cloud.png'/>


Database on the Federated Cloud
-------------------------------
The next step is to distribute transactional databases across the federated cloud.  In this model,
* database user will get similar performance from each cloud provider and
* resilience will be improved.

While this is easier said than done as distributing a transactional database is very difficult mainly due to network
latency, solutions exist that either
* relax the consistency constrains with new database models, for example using a NoSQL database or
* propose improved clustering implementation of existing relational databases.

The first solution gets around the network latency constraints but requires new applications to be written following
the new model and existing one to be rewritten.  The second solution still requires network latency to be as low as
possible and a design that limits transactional requests to the minimum. In practice, applications deployed on
the federated cloud will use the first solution as much as possible and the second when absolutely necessary.

One of the main advantage of using distributed databases on the federated cloud is symmetry.  No provider is special.
In the diagram below, we could easily add a fifth provider and simply connect the new database node to the existing
ones.

<img class="diagram" alt='Long Feedback Loop' src='/assets/drawings/federated-db/with-federated-database.png'/>

Benefits
========
Using a distributed database on the federated cloud gives
* improved reliability,
* provider independence because there is no a privileged provider hosting the database,
* simplified deployment thanks to symmetry.

Cost
====
The related costs are the following:
* keeping relational database performance is more costly when distributed,
* development cost increases when using a relaxed consistency model.

Conclusion
==========
This small article presented how the cloud influenced IT infrastructure and how the federated cloud will be used to
avoid vendor locking and improving reliability.  We took the view that distributed database, either relational or not,
are well suited in this context.  In the second part, we will list existing providers that fit this our model.

