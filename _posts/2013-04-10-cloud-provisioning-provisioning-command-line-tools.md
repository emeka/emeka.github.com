---
layout: post
title: "Open Source Cloud Service Provisioning Command Line Tools"
tags : [DevOps, operation, provisioning, cloud, BOSH, Juju, Whirr, Pallet ]
tweet: What are the current open source cloud service provisioning tools?
hashtags: #devops #cloud
bitmark: http://bit.ly/ZfhQMy
---
{% include JB/setup %}

Introduction
============
This post gives a survey of available open source cloud service provisioning command line tools.  We
talk about provisioning full services and not individual pieces of cloud infrastructure, and we take the angle that
command line tools come before any pretty management and monitoring application because the latter should be
provisioned by the same tools as your services.

**Note:**  This is a work in progress and this post will be augmented as time goes. Your comments are welcome as we have
certainly overlooked some projects.

Service
=======
Services is our units of deployment. They are building blocks that carry out well defined tasks. Several services
can be put together to participate in a business process. Services are independently deployed following their own
release schedule. When necessary, services can be grouped and deployed together.

Service is where we deliver value and therefore service must be a first class concept in any provisioning tools.

Criteria
========

Here are our initial criteria:
* we should be able to provision a multi node service on the cloud executing one command line,
* the tool must be compatible with Amazon Web Services (AWS), OpenStack and VMWare,
* the configuration must be file based and not buried in a database,
* it must be an active open source project.

Native Service Provisioning
=========================
Here are the tools that natively include the concept of service in their data model:

* [BOSH](https://github.com/cloudfoundry/bosh) from the [Cloud Foundry](http://www.cloudfoundry.com) project, find the
documentation [here](http://cloudfoundry.github.io/docs/running/bosh),
* [Juju](https://juju.ubuntu.com/) from [Canonical](canonical.com) with the documentation
[here](https://juju.ubuntu.com/docs/),
* [Whirr](http://whirr.apache.org/) from the [Apache Hadoop](http://hadoop.apache.org/) project, a service is called
a cluster.

Surprisingly, very few open source provisioning projects have a first class service concept in their data model.

Node Provisioning
=================
The following tools can be used to provision services but do not have a native concept of service in their data
model.  However, their configuration semantic is rich enough to express it with some naming conventions:

* [Puppet](puppetlabs.com): we can start VMs, install puppet and execute puppet code to either create a
puppet server and one or more clients, or configure nodes without a central puppet server.  The concept of service can be added using a service property and using puppet environment to isolate different
services from each other.
* [Chef](http://www.opscode.com/): Using [knife plugins](http://docs.opscode.com/plugin_knife.html) to start a node on
different IaaS and a mix of [knife-server](https://github.com/fnichol/knife-server) to bootstrap a chef server, and
[knife boostrap](http://docs.opscode.com/knife_bootstrap.html) to start and configure clients.
* [Pallet](http://palletops.com/) a clojure command line tool build on top of the [jclouds](http://www.jclouds.org/)
library.
* [Salt Stack](saltstack.com) see OpenCredo [post](http://www.opencredo.com/blog/a-dive-into-salt-stack) about it.
* [CFEngine](http://cfengine.com/).

Related Tools
=============
This section lists some related tools worth mentioning.

Infrastructure Management
-------------------------

* [delta cloud](http://deltacloud.apache.org/) is not a command line tool but a server providing a
[REST API](http://deltacloud.apache.org/rest-api.html).
Admin Manazine has a good [article](http://www.admin-magazine.com/Articles/Many-Clouds-One-API) about it.

Libraries
----------
Here are the libraries underlying most of the provisioning tools described above:
* [jclouds](http://www.jclouds.org/), a java library,
* [dasein](https://github.com/greese/dasein-cloud), an other java library,
* [Fog](http://fog.io), a ruby library and
* [Apache Libcloud](http://libcloud.apache.org/) a Python library.

Conclusion
==========
We have given the current list of open source cloud service provisioning command line tools.  We have insisted on having
a first class service concept because services deliver the value.  We have found
that very few open source tools have it.


