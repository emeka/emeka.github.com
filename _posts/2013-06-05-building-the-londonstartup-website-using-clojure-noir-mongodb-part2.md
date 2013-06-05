---
layout: post
title: "Building the London Startup Directory website using Clojure, Noir and MongoDB on Heroku - Part Two"
tags : [London Startup, Clojure, Noir, MondoDB, Heroku]
tweet: Take a look at @EmekaMosanya experience building a site with Clojure, Noir and MongoDB
hashtags: #clojure #noir #mongodb
bitmark:
---

{% include JB/setup %}

Introduction
============
This is the second installment of my foray into website development using Clojure, Noir and MongoDB.
Please see the read
[Part One](http://emekamosanya.com/2013/05/29/building-the-londonstartup-website-using-clojure-noir-mongodb-part1/)
if you missed it.

In this post, I will focus on MongoDB integration.

The code is a work in progress available on [github](http://github.com/emeka/londonstartup)

Local Install
=============
First, we need to be able to develop and test our application locally.

    brew install mongodb
    mongod

This installs and runs MongoDB using the default port and storing data under */usr/local/var/mongodb*.

The following commands will test that your database is behaving as expected.

    mongo
    > db.test.save( { a: 1 } )
    > db.test.find()

Monger
======

I use [Monger](http://clojuremongodb.info/) which requires the following dependency in your project.clj:

    [com.novemberain/monger "1.5.0"]

Connection to the database is configured in the *src/londonstartup/models.clj* file like this:

    (ns londonstartup.models
      (:require [monger.core :as mg]))


    (defn initialize []
      (let [uri (get (System/getenv) "MONGOLAB_URI" "mongodb://127.0.0.1/londonstartup")]
        (monger.core/connect-via-uri! uri)))

We use the *MONGOLAB_URI* environment variable. This is the key to connect to a different database
in each environment if you are using Mongolab services on Heroku.  You will need to adapt this if you are using MongoHQ
for example.  By default, we connect to a local MongoDB instance. The database name
is set to be "londonstartup".

The initialisation is called in the modified *src/londonstartup/server.clj* file

    (ns londonstartup.server
      (:require [noir.server :as server]
                [londonstartup.models :as models]))

    (server/load-views-ns 'londonstartup.views)

    (defn -main [& m]
      (let [mode (keyword (or (first m) :dev))
            port (Integer. (get (System/getenv) "PORT" "8080"))]
        (models/initialize)
        (server/start port {:mode mode
                            :ns 'londonstartup})))

Model
=====
Our application is the London Startup Directory which simply lists London startups,
as you have certainly figured out. The main object is a startup,
which looks like this in JSON:

    { "_id" : ObjectId("51adbfdcb0c606762558d6c0"),
      "website" : "www.example.com",
      "name" : "Example" }

Website and name must be unique.

We need the standard CRUD functions plus the ability to list all startups. There is also some validation code
to ensure that each startup object is well formed.  Here is the
[code](https://github.com/emeka/londonstartup/blob/master/src/londonstartup/models/startup.clj):

    (ns londonstartup.models.startup
      (:require [monger.core :as mg]
                [monger.collection :as mc]
                [noir.validation :as validate]
                [londonstartup.models :as models])
      (import org.bson.types.ObjectId))


    (def ^:dynamic collection "startups")

    ;; Result
    (defn add-error [result key msg]
      (let [errors (get-in result [:errors key] [])]
        (assoc-in result [:errors key] (conj errors msg))))

    (defn result [value]
      {:value value})

    (defn error [key msg]
      {:errors {key [msg]}})

    (defn has-error? [result]
      (contains? result :errors ))

    (defn errors [result]
      (:errors result))

    (defn value [result]
      (:value result))

    ;; Validation
    (defn has-id [startup]
      (let [id (:_id startup)]
        (and id (not-empty (str id)))))

    (def validation-rules
      [[:website validate/has-value? "A startup must have a website"],
       [:name validate/has-value? "A startup must have a name"]])

    (defn valid?
      ([startup]
        (reduce #(valid? startup %1 %2) (result startup) validation-rules))
      ([startup result [field test msg]]
        (if (test (field startup))
          result
          (add-error result field msg)
          )))

    ;; CRUD

    (defn total []
      (result (mc/count collection)))

    (defn id->startup [id]
      (result (mc/find-one-as-map collection {:_id id})))

    (defn website->startup [website]
      (result (mc/find-one-as-map collection {:website website})))

    (defn website-free? [website]
      (result (= 0 (mc/count collection {:website website}))))

    (defn startups []
      (result (mc/find-maps collection)))

    (defn add! [startup]
      (let [validation-result (valid? startup)]
        (if (not (has-error? validation-result))
          (let [oid (if (has-id startup) (:_id startup) (ObjectId.))]
            (if (nil? (value (id->startup oid)))
              (if (value (website-free? (:website startup)))
                (result (get (mc/insert-and-return collection (merge startup {:_id oid})) :_id ))
                (error :website "Website already in use."))
              (error :startup "Startup already exists")))
          validation-result)))

    (defn update! [startup]
      (let [validation-result (valid? startup)]
        (if (not (has-error? validation-result))
          (when-let [id (:_id startup)]
            (when-let [old-startup (value (id->startup id))]
              (if (or (= (:website startup) (:website old-startup)) (value (website-free? (:website startup))))
                (do
                  (mc/update-by-id collection id startup)
                  (result id))
                (error :website "Website already in use."))))
          validation-result)))


    (defn remove! [id]
      (result (mc/remove-by-id collection id)))

    (defn remove-website! [website]
      (result (mc/remove collection {:website website})))

Let's go through it.

Result
------

I do not want to use the
<code>noir.validation/rule</code> here as it would mean that view code is bleeding in my model layer.
Therefore, the functions return <code>result</code> maps instead of directly returning
result values.

The result holds the value and optional error messages and a number of helpers allow you to manipulate it:

    (result "a result")
    ;;{:value "a result"}

    (value (result "a result"))
    ;;"a result"

    (has-error? (result "a result"))
    ;;false

    (error :key "Error Msg")
    ;;{:error {:key ["Error Msg"]}

    (has-error? (error :key "Error Msg"))
    ;;true

    (add-error (result "a result") :key "Error Msg")
    ;;{:value "a result" :error {:key ["Error Msg"]}

There is certainly a better way to return errors, maybe with a state monad.  I am reluctant to use clojure
bindings as it is not functional enough for my taste.

Validation
----------

The validation code has been factored between the list of rules

    (def validation-rules
      [[:website validate/has-value? "A startup must have a website"],
       [:name validate/has-value? "A startup must have a name"]])

and the validation code

    (defn valid?
      ([startup]
        (reduce #(valid? startup %1 %2) (result startup) validation-rules))
      ([startup result [field test msg]]
        (if (test (field startup))
          result
          (add-error result field msg)
          )))

The validation code returns a result with the startup as value and the errors if any.

We can use the <code>noir.validation/has-value?</code> here because they are just predicate without side effect.

CRUD
----

Finally the CRUD code interacts with the database.  All functions are simple beside the <code>add!</code> and
<code>update!</code> which test for duplicate object and website names (Using a naive check, I admit).
The validation code in these two functions will be migrated to the validation rules in the future.

I have also defined the corresponding MongoDB collection in a dynamic var:

    (def ^:dynamic collection "startups")

This allow us to change the collection name during integration test.

Object ID
---------
Every MongoDB document (our unit of storage) requires an <code>_id</code>.  MongoDB will create a new document without
it but it means that our startup object is mutated.  The Monger website recommends to create the object id ourselves,
hence the following code in <code>add!</code>

    let [oid (if (has-id startup) (:_id startup) (ObjectId.))]

Test
----
You can find tests
[here](https://github.com/emeka/londonstartup/blob/master/test/londonstartup/models/startup_test.clj).  It is
currently a mix of unit and integration tests and you will need a running local database to execute them.

    (ns londonstartup.models.startup-test
      (:require [londonstartup.models.startup :as startup]
                [londonstartup.models :as models]
                [monger.core :as mg]
                [monger.collection :as mc])
      (:use clojure.test)
      (:use noir.util.test)
      (import org.bson.types.ObjectId))


    ;; Result test
    (deftest add-error
      (let [init-result (startup/result nil)
            result1 (startup/add-error init-result :website "Error1")
            result2 (startup/add-error result1 :website "Error2")
            result3 (startup/add-error result2 :name "Name Error")]
        (is (= {:value nil :errors {:website ["Error1"]}} result1))
        (is (= {:value nil :errors {:website ["Error1" "Error2"]}} result2))
        (is (= {:value nil :errors {:website ["Error1" "Error2"] :name ["Name Error"]}} result3))))

    (deftest result
      (is (= {:value nil} (startup/result nil)))
      (is (= {:value 3} (startup/result 3))))

    (deftest error
      (is (= {:errors {:website ["Error"]}} (startup/error :website "Error"))))

    (deftest has-error?
      (is (not (startup/has-error? (startup/result nil))))
      (is (startup/has-error? (startup/add-error (startup/result nil) :website "Error"))))

    (deftest errors
      (is (nil? (startup/errors (startup/result nil))))
      (is (= {:website ["Error"]} (startup/errors (startup/add-error (startup/result nil) :website "Error")))))

    (deftest value
      (is (nil? (startup/value (startup/result nil))))
      (is (= 3 (startup/value (startup/result 3)))))


    ;; CRUD test
    (let [google-id (ObjectId.)
          yahoo-id (ObjectId.)
          github-id (ObjectId.)
          google {:_id google-id :website "www.google.com" :name "Google Inc."}
          yahoo {:_id yahoo-id :website "www.yahoo.com" :name "Yahoo! Inc."}
          github {:_id github-id :website "www.github.com" :name "Github"}]

      ;; Fixtures
      (defn init-db [f]
        (models/initialize)
        (binding [startup/collection "startupsTEST"]
        (f)))

      (defn clean-db [f]
        (mc/remove startup/collection)
        (startup/add! google)
        (startup/add! yahoo)
        (f))

      (use-fixtures :once init-db)
      (use-fixtures :each clean-db)

      ;;Tests
      (deftest valid?
        (is (not (startup/has-error? (startup/valid? google))))
        (is (startup/has-error? (startup/valid? {}))))

      (deftest total
        (is (= 2 (startup/value (startup/total)))))

      (deftest id->startup
        (is (= google (startup/value (startup/id->startup google-id)))))

      (deftest website->startup
        (is (= google (startup/value (startup/website->startup "www.google.com")))))

      (deftest website-free?
        (is (startup/value (startup/website-free? "www.doesnotexist.com")))
        (is (not (startup/value (startup/website-free? "www.google.com")))))

      (deftest startups
        (is (= (list google yahoo) (startup/value (startup/startups)))))

      (deftest add!
        (startup/add! github)
        (is (= 3 (startup/value (startup/total)))))

      (deftest update!
        (is (= google-id (startup/value (startup/update! (merge google {:website "www.new.com"})))))
        ;(is (= "www.new.com" (:website (startup/value (startup/id->startup google-id))))))
        )

      (deftest remove!
        (startup/remove! google-id)
        (is (= 1 (startup/value (startup/total))))
        (is (= nil (startup/value (startup/id->startup google-id)))))

      (deftest remove-website!
        (startup/remove-website! "www.google.com")
        (is (= 1 (startup/value (startup/total))))
        (is (= nil (startup/value (startup/id->startup google-id))))))


The interesting part is the use of Clojure binding to change the database collection during test:

      ;; Fixtures
      (defn init-db [f]
        (models/initialize)
        (binding [startup/collection "startupsTEST"]
        (f)))

We can now safely run the test on the same database as the application knowing that we only touch test collections.

Running the test ensures that our code integrated correction with the database.

    lein test

    lein test londonstartup.models.startup-test

    Ran 16 tests containing 26 assertions.
    0 failures, 0 errors.

Heroku
======

After having tested that our model layer with a local database, it is time to configure our staging and production
environments.

Let's start by adding a MongoDB database.  I am using the MongoLab add-on but I could have used the MongoHQ.

    heroku addons:add mongolab:sandbox --remote staging
    heroku addons:add mongolab:sandbox --remote production

Even if the sandbox database (512MB) is free, you will need to have added you credit card information in
your Heroku account.

In your apps dashboard, you will find the MongoLab Sandbox add-on which will redirect you to the MongoLab website
where you will find the connection URL, username and password.  For example, the staging URL has the following
shape:

    mongodb://<dbuser>:<dbpassword>@ds12345.mongolab.com:35147/heroku_app987654321

This is the URL that you will use in your application.

Heroku will create a MONGOLAB_URI environment variable for you in each environment and you can verify it value using
the heroku command like this:

    heroku config
    === londonstartup-staging Config Vars
    JVM_OPTS:     -Xmx400m
    LEIN_NO_DEV:  true
    MONGOLAB_URI: mongodb://<dbuser>:<dbpassword>@ds12345.mongolab.com:35147/heroku_app987654321
    PATH:         .lein/bin:/usr/local/bin:/usr/bin:/bin

Since we use the MONGOLAB_URI environment variable in our code, we will automatically switch database
for each environment.

Now you can run same integration test remotely:

    heroku run lein test

    Running `lein test` attached to terminal... up, run.8765
    Picked up JAVA_TOOL_OPTIONS:  -Djava.rmi.server.useCodebaseOnly=true
    Retrieving org/clojure/tools.nrepl/0.2.3/tools.nrepl-0.2.3.pom from
    Retrieving org/clojure/pom.contrib/0.1.2/pom.contrib-0.1.2.pom from
    Retrieving clojure-complete/clojure-complete/0.2.3/clojure-complete-0.2.3.pom from clojars
    Retrieving org/clojure/tools.nrepl/0.2.3/tools.nrepl-0.2.3.jar from
    Retrieving clojure-complete/clojure-complete/0.2.3/clojure-complete-0.2.3.jar from clojars
    Picked up JAVA_TOOL_OPTIONS:  -Djava.rmi.server.useCodebaseOnly=true

    lein test londonstartup.models.startup-test

    Ran 16 tests containing 26 assertions.
    0 failures, 0 errors.

That's it!

Conclusion
==========
This post described my first steps using MongoDB with Clojure.  We have seen how to configure a local database and
write a model layer for a very simple application and test it. Finally, we have configured a MongoDB instance in our
Heroku environment and successfully run the same tests remotely.

In the next part, we will develop our application view.