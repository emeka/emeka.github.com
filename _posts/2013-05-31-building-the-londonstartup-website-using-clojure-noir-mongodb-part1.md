---
layout: post
title: "Building the London Startup Directory website using Clojure, Noir and MongoDB on Heroku - Part One"
tags : [London Startup, Clojure, Noir, MondoDB, Heroku]
tweet: Take a look at @EmekaMosanya experience building a site with Clojure, Noir and MongoDB
hashtags: #clojure #noir #mongodb
bitmark: http://bit.ly/17sSVhu
---

{% include JB/setup %}

Introduction
============
When I was looking for new opportunities in London, I found it difficult to get a list of the startups in the area.
There are many incomplete sources but no central up-to-date directory.

To experiment with [Clojure](http://clojure.org/), [Noir](webnoir.org),
[MongoDB](http://www.mongodb.org/) and [Heroku](heroku.com), I decided to quickly throw and new website doing just that:
A curated London Startup Directory.

Getting the list of startup in London or in any location is a moving target.  By definition, startups appear and
disappear all the time.  The service must be able to dynamically keep itself up to date either by regularly asking
registrar to confirm their data or by listening the interworld to check if the startup is still alive.

In part one, I will focus on creating a staging and production environment on Heroku to run a skeleton of the new
website.  Part two will concentrate on the MongoDB connection.

Noir Toolset
============
Install [Leiningen](https://github.com/technomancy/leiningen):

    brew install leiningen

I had to manually add the Noir template in my *~/.lein/profile.clj*:

    {:user {:plugins [[noir/lein-template "1.3.0"]]}}

This allowed me to create my Noir project scaffold:

    lein noir new londonstartup
    cd londonstartup

You can now start the server in the [REPL](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)

    lein repl

    (server/start 8080)

The server/start command will start the server on port 8080 of your local machine:

    Starting server...
    2013-05-29 15:03:29.808:INFO::Logging to STDERR via org.mortbay.log.StdErrLog
    2013-05-29 15:03:29.809:INFO::jetty-6.1.25
    2013-05-29 15:03:29.821:INFO::Started SocketConnector@0.0.0.0:8080
    Server started on port [8080].
    You can view the site at http://localhost:8080

Opening localhost:8080/welcome with return a simple default page.  Editing a file in your project is automatically
picked up.  The development loop is short.

Github
======

Let's create a new local git repository

    git init
    git add .
    git commit -m "Initial Commit."

Create a new remote [github.com/emeka/londonstartup](https://github.com/emeka/londonstartup) repository on Github
and push:

    git remote add origin git@github.com:emeka/londonstartup.git
    git push -u origin master


Heroku
======

Install [Heroku Toolbelt](https://toolbelt.heroku.com/) (I prefer to use homebrew):

    brew install heroku-toolbelt

This should give you the *heroku* command.  If you read the
[Getting Started with Clojure on Heroku](https://devcenter.heroku.com/articles/clojure) document, you will see that
Heroku has its own Leiningen template which add some extra convenience features useful for Heroku development.

Since we already have created our project, we will add these features manually.

The modified project.clj looks as following:

    (defproject londonstartup "0.1.0-SNAPSHOT"
                :description "London Startup Directory"
                :dependencies [[org.clojure/clojure "1.4.0"]
                               [noir "1.3.0-beta3"]]
                :license {:name "Simplified BSD License"
                          :url "http://en.wikipedia.org/wiki/BSD_licenses"}
                :main londonstartup.server
                :min-lein-version "2.0.0"
                :plugins [[environ/environ.lein "0.2.1"]]
                :hooks [environ.leiningen.hooks]
                :profiles {:production {:env {:production true}}})

In addition, the Heroku template adds a Procfile in the root directory of our project:

    echo "web: lein with-profile production trampoline run -m londonstartup.server" > Procfile

This file is used by *foreman* to start a local server:

    PORT=8888 foreman start

should now start a server accessible at [http://localhost:8888](http://localhost:8888).  I needed to specify the port
because the default one conflicts with launchd.

We will do the right thing from the start and create a staging environment where the application is validated before
going to production. The document
[Managing Multiple Environments for an App](https://devcenter.heroku.com/articles/multiple-environments) explains how
to configure several environments.

    heroku create londonstartup-staging --remote staging

creates a new Heroku app on the remote branch call staging.

    git add .
    git commit -m "..."
    git push staging master

After some seconds, my new Heroku application was available on
[londonstartup-staging.herokuapp.com](http://londonstartup-staging.herokuapp.com).

    git config heroku.remote staging

will make the staging remote Heroku's default and will prevent us to touch production by mistake.

Let's directly create the production environment:

    heroku create londonstartup-production --remote production
    git push production master

We now have a production environment at
[londonstartup-production.herokuapp.com](http://londonstartup-production.herokuapp.com)

I like to have a staging and production branches that will track staging/master and production/master:

    git config push.default tracking
    git checkout -b staging --track staging/master
    git checkout -b production --track production/master

Now, any push from staging or production will update my staging or production environment respectively.  I still have
to setup a proper tagging convention to only push well tagged commit to staging and production.

Conclusion
==========

Et voilà, we have two environments deployed plus our github repository ready for development.  Part two will focus on
MongoDB configuration and a minimal functionality. We will later add an automatic CI.






