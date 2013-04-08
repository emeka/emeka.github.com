---
layout: post
title: "SOOP: Service Oriented Operation and Provisioning (Part one)"
tags : [puppet, SOOP, DevOps, operation, provisioning, cloud, Financial Times]
tweet: Take a look at #SOOP, Service Oriented Operation and Provisioning (Part one)
hashtags: #SOOP #puppet #devops #cloud
bitmark:
---
{% include JB/setup %}

Introduction
============

This post is the first of a several articles describing a model of software deployment called *Service Oriented
Operation and Provisioning* (SOOP). This model has been developed while working at the Financial Times in London to
solve some of their service delivery problems.

In part one, I will describe the problems that we were trying to solve and the model that was developed to solve them.
Part two will focus on the implementation, called FT Cloud, and how it was received by developers and infrastructure
teams. Part three will present some thoughts about the model and will propose some improvements.

It is important to make a clear distinction between the model and its implementations. No implementation can save a
flawed model but a good model can be implemented in many ways. This model is far from perfect but is a first step in the
right direction.

The Objective
=============

Our objective was to reduce the *cycle time* between business requirements and product delivery to customers.
Here we use the concept of cycle time as described in [\[1, p. 62\]][2], where time is the universal currency:

*"Everything that goes wrong in a process shows up as a time delay. Defects add delay. Complexity slows things down.
Low productivity shows up as taking more time. Change intolerance makes things go very slowly. Building the wrong thing
adds a huge delay. Too many things in process creates queues and slows down the flow. Time, specifically cycle time, is
the universal lean measurement that alerts us when anything is going wrong. A capable development process is one that
transforms identified customer needs into delivered customer value at a reliable, repeatable cadence, which we call
cycle time. It is this cycle time that paces the organization, causes value to flow, forces quality to be built into the
product, and clarifies the capacity of the organization."*

When we started to look at our cycle time at the end of 2010, the result was not very good. We were only able to release
to production every four to six weeks and it was a major endeavour which often tied several people for a full sprint
with the accompanying pain and tears.

We knew that we could do better.

The Problems
============

The problems were multiple and could be classified in two categories:
* organizational frictions and
* long feedback loop.

The sections below will address each of these concerns separately.

Organizational Frictions
------------------------

The first problem was one of organizational frictions due to a misalignment of priorities and the dilution of
responsibilities between teams.

In our case, it was the classic divide between operation and product teams where:
* product teams are close to the business and responsible for developing and delivering new products fast,
* operation teams are further away from the business and responsible for existing product availability and new product
platform creation.

Their priorities are not aligned or even in opposition but they share the product delivery responsibilities since
operation owns the platform on which products are deployed. This is source of tension.

The result was the situation illustrated in the diagram below where
the operation teams acted as gates during
product delivery. Each time a product had to go through a gate, precious time and energy was lost in
* communication (and misunderstanding),
* task switching and
* delays.

Since the operation teams must serve several product teams, they are an obvious bottleneck.

<img class="diagram" alt='Organizational Frictions Diagram' src='/assets/drawings/soop/organizational-friction.png'/>

But there is another side to this story: as product teams did not take full responsibility for product delivery and
availability, they had a tendency to throw things over the wall, brushing over non functional requirements like
monitoring, disaster recovery or simply operational documentation. This in turn was feeding mistrust in operation teams.

It must be emphasised that the inefficiencies are not due to teams or individual competencies. Individuals in each team
were all highly qualified professionals with an excellent knowledge of their core specialities. The problems described
here are the result of a certain organizational structure, i.e. ownership, responsibility and control.

Long Feedback Loop
------------------

The second problem was the long feedback loop where defects were detected too late. Finding a defect at a developer's
workstation is always less expensive than discovering that same defect further down the delivery pipeline. A defect
in production can have far reaching consequences and defects were unfortunately often found at this ultimate stage.

<img class="diagram" alt='Long Feedback Loop' src='/assets/drawings/soop/long-feedback-loop.png'/>

The reasons for such a long feedback loop were the following:
* manual and non repeatable configurations,
* no environment identical to production before reaching production,
* not enough environments available.

As a result, each deployment to production was an adventure full of surprises.

The Vision
==========
In our ideal world, each product team has full responsibility for their product development, deployment and
maintenance, and it is able to create the required infrastructure and deploy to it automatically.

To achieve this, we had to remove the gates and replace them with automation. The product teams choose or develop the
right automation for their specific needs. The operation teams either provide design patterns and specialised advice to
product teams, or they develop reusable automation libraries. In any case, product teams must have the freedom to choose
since they are ultimately responsible.

In the future, it will be more and more likely that the right tool for the job will be already available from an
external provider.

<img class="diagram" alt='Organizational Vision Diagram' src='/assets/drawings/soop/organizational-vision.png'/>

To reduce the feedback loop, we applied the Continuous Delivery principles [\[2, p. 113\]][2][\[3\]][3]:
* Only Build Your Binaries Once (Every build is a candidate for release),
* Deploy the Same Way to Every Environment (Do it automatically),
* Smoke-Test Your Deployments,
* Deploy into a Copy of Production,
* Each Change Should Propagate through the Pipeline Instantly,
* If Any Part of the Pipeline Fails, Stop the Line.

As a result, we will release with confidence and more often. Delivering more often shortens the feedback loop between
our customers and our business resulting in products and services that actually fulfil our clients' needs.

The Model
=========
**Note:** We describe the model as implemented at the Financial Times. The implementation will be described
in part two of these articles. In part three, we will examine the lessons drawn from this experience and potential
improvements.

Entities
--------

Here are our entities:

* *Services* are the units of deployment. They are building blocks that carry out well defined tasks. Several services
can be put together to participate in a business process. Services are independently deployed following their own
release schedule. When necessary, services can be grouped and deployed together.

* *Service definitions* are versioned artifacts that encapsulate everything required to deploy services. Knowing a service
definition name and its version must be enough to deploy a service with a simple command line. Services definitions are
composed of service components and node definitions.

* *Service components* are the building blocks of a service definition, for example a database or an application server
configuration required to support service delivery.

* *Environments* are isolated infrastructure resources like CPU, network and storage, with specific capacity, reliability,
accessibility and security requirements, for example we can have development, test and production environments.

* *Domains* are sets of *nodes* that collaborate with each other to provide one or more services.
Domains have unique ids and belong to an environment.

* *Domain Definitions* configure the number of nodes, their roles and the service definitions that are deployed in
a domain. The domain definition can also override any default service configuration contained in the
service definitions.

* *Nodes* are any network-connected virtual and physical machines. Nodes have unique names.
Services are delivered by one or more node(s). Node are not shared between services.

* *Node definitions* are mappings between service nodes and service components.

Service Definition Details
--------------------------

The service definition artifact is the cornerstone of the SOOP model. It must encapsulate everything required to
deploy a service including any application binaries, database schema and default configuration. The service definition
is the blueprint of a service and can be the result of a build tool that gathers all the required components and packs
them. It is likely that service components will themselves be composed of building blocks provided by libraries.

<img class="diagram" alt='Organizational Vision Diagram' src='/assets/drawings/soop/service-definition-details.png'/>

Since the service definition is an artifact, it can be stored and exchanged. It can also participate in a continuous
delivery pipeline and depending on the acceptance criteria, it will reach production unchanged or it will be discarded.
In addition, each service definition is built only once and deployed the same way everywhere.

Service Deployment
==================

A deployment tool will interpret a domain definition, create the nodes if necessary, associate them with their role
and start the configuration process. The configuration process will read the node definitions and configure the
corresponding service components. Services are ready for use when all nodes are configured.

Since services are well defined by their versioned artifact, it is easy to know exactly what has been deployed in a
domain. Testers know, for example, that they are testing an instance of the access-service-3.2.3.

<img class="diagram" alt='Organizational Vision Diagram' src='/assets/drawings/soop/deploy-services-into-domains.png'/>

Domains act as a naming scope as a service definition can be deployed several times in the same environment, for example
in a development environment where each developer can create their own domain. On the other hand, the same service
cannot be deployed more than once in the same domain.

Service deployment tools should be able to deploy the same service definition several times in the same domain and
they should be able to upgrade a service using a new service definition version. Depending on service specifics like
database migration schema, service downgrades can be handled by deployment tools or by the underlying virtualization
platform, for example using virtual machine snapshots.

Benefits
========
By encapsulating in one versioned artifact everything required to deploy a service, we enjoy the following benefits:
* there is no manual configuration,
* we deploy in the same way everywhere from a local workstation up to production,
* we know what we are testing,
* problems are discovered earlier,
* we trust our deployment since it has been tested many times,
* everybody (testers, developers, operations, trainers) can deploy a
service without knowing the deployment details,
* product teams can control deployment,
* operation teams can review the deployment code,
* knowledge is progressively documented in code,
* we can reuse configuration building blocks and create platforms, and

... the list goes on.

Cost
=========
SOOP has the following costs associated to it:
* it requires people to change their habits,
* automation can take some time to get right and there is a setup cost,
* operation teams will need training to move towards an automation-developer role,
* product teams will need training to understand infrastructure better.
* people will resist change as they will feel threatened in their role.

However the benefits largely outweigh the costs very quickly after the initial learning curve.

Conclusion
==========
This article describes a deployment model called Service Oriented Operation and Provisioning (SOOP). Its originality
lies in the use of versioned service definition artifacts that encapsulate everything required to deploy services.
After the initial learning curve, using SOOP allows product teams to have full control and responsibility for their
product delivery while operation teams focus on their specialised added value. As a result, SOOP will cut delivery cycle
time by reducing organizational frictions, shortening the feedback loop and automating deployment.
Reducing cycle time brings customers closer to the business and results in better products for the benefit of all.


References
==========
\[1\]: [Implementing Lean Software Development From Concept to Cash, Mary Poppendieck and Tom Poppendieck, Addison-Wesley Professional, 2006][1]<br/>
\[2\]: [Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation, Jez Humble and David Farley, Addison-Wesley Signature, 2010][2]<br/>
\[3\]: [Continuous Delivery: Anatomy of the Deployment Pipeline,  Jez Humble and David Farley, InformIt, 2010][3]

[1]: http://my.safaribooksonline.com/0321437381 "Implementing Lean Software Development From Concept to Cash"
[2]: http://my.safaribooksonline.com/9780321670250 "Continuous Delivery: Reliable Software Releases through Build, Test, and Deployment Automation"
[3]: http://www.informit.com/articles/article.aspx?p=1621865&seqNum=3 "Continuous Delivery: Anatomy of the Deployment Pipeline"