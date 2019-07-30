# M220JS Below are my own notes How to use this guide.
This guide will not really be broken into Chapters but broken up by concepts. If a later chapter has information that furthers conceptual knowledge of a topic
it will be written up to the same section. In the future you might see UML code which is a text based way to draw diagrams in a UML supported editor.
I highly recommend Boostnote to view this markdown.

## Using the Course without Cloud DBs
* As a user I want to complete this course without using my cloud or using the local version of the db.

## Overview

The instructions here will both be imperative and declarative. It wont have any answers to the course. It will just be a collection of thoughts and deep dives into the code.

* [Understanding Docker as if it were a Game Boy - James Audretsch - Medium](https://medium.com/@audretschjames/understanding-docker-as-if-it-were-a-gameboy-96c96392efbf)
* [GitHub - dylanlrrb/Please-Contain-Yourself: A Docker tutorial written for people who don't actually know Docker already.](https://github.com/dylanlrrb/Please-Contain-Yourself/)

* If these two articles had a baby, this would be it.

## Pre-stuff
* download mflix from the handouts
* unzip it. and go to that folder
* npm install

## Docker Version

* Get the image
  * docker pull mongo:4.04 or just pull the latest
* run the image with a few settings
  * ```bash
    docker run -d -p 27018:27017 /
    --name m220js /
    --v $(pwd)\input:\db\configdb /
    --v course-m220js-data:\db\data /
    mongo
    ```
    
  * -p <host_port>:<container_port> — map host port to container (publish)
      * that means my computer\'s port (host) will transfer any requests over to 27017 on the container. I have mongodb installed and the default port is already taken over by 27017.
    * -d means detached so you don\'t get weird logging and you can still use the terminal afterwards
    * --v [lhs:rhs] (save the data so your container can go down but this memory card stays in place)
      * left hand side is where it goes to. Either you name it like i did with course-m220js-data
        * that gets stored inside `docker volume ls`
      * specify a directory path in the lhs if you dont want docker to store the volume.
      * \db\configdb are for storing info for sharded clusters
        * https://stackoverflow.com/questions/56855283/what-is-data-configdb-path-used-for-in-mongodb
      * \db\data are for database info itself
* ```
  mongorestore --gzip --uri --drop \
  mongodb://localhost:27018 data
  ```
  * load into our mongo database everything from the data folder using gzip. Since the folder inside of data is called mflix, your db name is also called **mflix**
    * ---drop means drop any database called mflix if it finds it then mongorestore (load the data);
    * confirm: 
      ```bash
      mongo mongodb://localhost:27018
      show dbs;
      use mflix;
      show collections;
      ```
* Go to your app's root directory
* the next step is to run the app's server 
  * checkout out what npm start does.
  * ` "start": "nodemon -L ./index.js",`
    * -L means that if i had mounted my app in a container, this -L shit will make it work.
    * Now let's checkout index.js
    * it has [@babel/register · Babel](https://babeljs.io/docs/en/babel-register) which transpiles any es5 stuff on the fly.
    * the exports export everything from the src file { all src stuff }. this is a little crazy that all functions are activated when you export them.       



Notes:

When you stop a docker container from spinning, the volume is cached cuz docker is insane. Either do this or add the `--rm` when you spin up the container.
[Docker — Clean Up After Yourself! - Yohan Liyanage - Medium](https://medium.com/yohan-liyanage/docker-clean-up-after-yourself-8f39bfbd2000)

## Aggregation applied to a cursor

Vocabulary
* def: Cursor
  * A pointer to the result set of a query. Clients can iterate through a cursor to retrieve results. ... Essentially, the cursor is the conduit you will use to extract all the results of a query from the database.Sep 26, 2014 
* Aggregation
  * def: Aggregation
    * Aggregation groups the data from multiple documents and operates in many ways on those grouped data in order to return one combined result. In sql count(*) and with group by is an equivalent of MongoDB aggregation. ... Let us now see how to make use of the aggregate function in MongoDB.
  * Aggregation is a pipeline
    * pipelines are composed of stages (broad unit of work)
    * stages
      * a specific unit of work inside stages is called 'expressions'
      * expressions are functions
    * example
      * documents enter the assemply line
      * Each stage does work on this document
      * output then is the final form.

limit
sort
skip

$limit, $sort, $skip
```js
const skippedPipeline = [
      { $match: { directors: "Sam Raimi" } },
      { $project: { _id: 0, year: 1, title: 1, cast: 1 } },
      { $sort: { year: 1 } },
      { $skip: 5 },
    ]
    
movies.aggregate(skippedPipeline)
```
is equivalent to

```js
 const skippedCursor = movies
      .find({ directors: "Sam Raimi" }, { _id: 0, year: 1, title: 1, cast: 1 })
      .sort([["year", 1]])
      .skip(5)
```


original MD below
=====
Mflix
=====

This is a short guide on setting up the system and environment dependencies
required for the MFlix application to run.


Project Structure
-----------------

Downloading the **mflix-js.zip** handout may take a few minutes. Unzipping the
file should create a new directory called **mflix-js**.

Most of your work will be implementing methods in the **dao** directory, which
contains all database interfacing methods. The API will make calls to Data
Access Objects (DAOs) that interact directly with MongoDB.

The unit tests in **test** will test these database access methods directly,
without going through the API. The UI will run these methods in integration
tests, and therefore requires the full application to be running.

The lesson handouts can be found in the **test/lessons** directory. These files
will look like **<lesson-name>.spec.js**, and can be run with ``npm test -t
<lesson-name>``.

The API layer is fully implemented, as is the UI. The application is programmed
to  run on port **5000** by default - if you need to run on a port other than
5000, you can edit the **dotenv_win** (if on Windows) or the **dotenv_unix** file
(if on Linux or Mac) in the root directory to modify the value of **PORT**.

Please do not modify the API layer in any way, under the **mflix-js/src/api**
directory. This may result in the front-end application failing to validate some
of the labs.


Node Library Dependencies
-------------------------

The dependencies for the MFlix application should be downloaded using the
``npm`` command-line tool. You can get this tool by `downloading Node.js
<https://nodejs.org/en/download/>`_. Make sure to choose the correct option for
your operating system.

Once the installation is complete, you may need to restart your computer before
using the command line tools. You can test that it's installed by running the
following command:

.. code-block:: sh

  node -v

This should print out the version of ``node`` you currently have - we recommend
using version 10 or later, so this command should print something like
``v10.x``.

Once ``npm`` is installed, you can install the MFlix dependencies by running the
following command from the **mflix-js** directory:

.. code-block:: sh

  npm install

You must run this from the top level of the project, so ``npm`` has access to
the **package.json** file where the dependencies are.

You may see warnings depending on your operating system from **fsevents** or
Husky warning about **git** missing. These are informational only and do not
impact the functionality of the application. You can safely ignore them.


MongoDB Installation
--------------------

It is recommended to connect MFlix with MongoDB Atlas, so you do not need to
have a MongoDB server running on your host machine. The lectures and labs in
this course will assume that you are using an Atlas cluster instead of a local
instance.

That said, you are still required to have the MongoDB server installed, in order
to be able to use two server tool dependencies:

- ``mongorestore``

  - A utility for importing binary data into MongoDB.

- ``mongo``

  - The MongoDB shell

To download these command line tools, please visit the
`MongoDB download center <https://www.mongodb.com/download-center#enterprise>`_
and choose the appropriate platform.

All of these tools are free to use. MongoDB Enterprise is also free to use for
testing and evaluation purposes.


MongoDB Atlas Cluster
---------------------

MFlix uses MongoDB to persist all of its data.

One of the easiest ways to get up and running with MongoDB is to use MongoDB
Atlas, a hosted and fully-managed database solution.

If you have taken other MongoDB University courses like M001 or M121, you may
already have an account - feel free to reuse that cluster for this course.

*Note: Be advised that some of the UI aspects of Atlas may have changed since
the inception of this README, therefore some of the screenshots in this file may
be different from the actual Atlas UI interface.*


Using an existing MongoDB Atlas Account:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you already have a previous Atlas account created, perhaps because you've
taken one of our other MongoDB university courses, you can repurpose it for
this course.

Log-in to your Atlas account and create a new project named **M220** by clicking
on the **Context** dropdown menu:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_create_project.png

After creating a new project, you need to create an **mflix** free tier cluster.


Creating a new MongoDB Atlas Account:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you do not have an existing Atlas account, go ahead and `create an Atlas
Account <https://cloud.mongodb.com/links/registerForAtlas>`_ by filling in the
required fields:

.. image:: https://s3.amazonaws.com/university-courses/m220/atlas_registration.png


Creating a free tier cluster called "mflix":
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*Note: You will need to do this step even if you are reusing an Atlas account.*

1. After creating a new project, you will be prompted to create the first
   cluster in that project:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_create.png


2. Choose AWS as the cloud provider, in a Region that has the label
   **Free Tier Available**:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_provider.png


3. Select **Cluster Tier** M0:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_tier.png


4. Set **Cluster Name** to **mflix** and click **Create Cluster**. It may take
   7-10 minutes to successfully create your Atlas cluster:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_name.png


5. Once you press **Create Cluster**, you will be redirected to the account
   dashboard. In this dashboard, make sure you set your project name to
   **M220**. Go to **Settings** menu item and change the project name from the
   default **Project 0** to **M220**:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_project.png


6. Next, configure the security settings of this cluster, by enabling the **IP
   Whitelist** and **MongoDB Users**:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_ipwhitelisting.png

Update your IP Whitelist so that your app can talk to the cluster. Click the
**Security** tab from the **Clusters** page. Then click **IP Whitelist**
followed by **Add IP Address**. Finally, click **Allow Access from Anywhere**
and click **Confirm**.

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_allowall.png


7. Then create the application MongoDB database user required for this course:

  - username: **m220student**
  - password: **m220password**

You can create new users through **Security** -> **Add New User**.

Allow this user the privilege to **Read and write to any database**:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_application_user.png


8. When the user is created, and the cluster deployed, you have the option to
   ``Load Sample Dataset``. This will load the Atlas sample dataset, containing
   the MFlix database, into your cluster:

.. image:: https://s3.amazonaws.com/university-courses/m220/load_sample_dataset.png

**Note: The MFlix database in the Sample Dataset is called "sample_mflix".**


9. Now you can test the setup by
   connecting via the Mongo shell. You can find instructions to connect in the
   **Connect Your Application** section of the cluster dashboard:

.. image:: https://s3.amazonaws.com/university-courses/m220/cluster_connect_application.png

Go to your cluster **Overview** -> **Connect** -> **Connect Your Application**.
Select the option corresponding to your local MongoDB version and copy the
``mongo`` connection command.

The below example connects to Atlas as the user you created before, with
username **m220student** and password **m220password**. You can run this command
from your command line:

.. code-block:: sh

  mongo "mongodb+srv://m220student:m220password@<YOUR_CLUSTER_URI>"

By connecting to the server from your host machine, you have validated that the
cluster is configured and reachable from your local workstation.

You may see the following message when you connect::

  Error while trying to show server startup warnings: user is not allowed to do action [getLog] on [admin.]

This is a log message, **not** an error - feel free to ignore it.


Running the Application
-----------------------

In order for the application to use Atlas, you will need a file called **.env**
to contain the connection information. In the **mflix-js** directory you can
find two files, **dotenv_unix** (for Unix users) and **dotenv_win** (for Windows
users).

Open the file for your chosen operating system and enter your Atlas SRV
connection string as directed in the comment. This is the information the driver
will use to connect. Make sure **not** to wrap your Atlas SRV connection between
quotes::

  MFLIX_DB_URI = mongodb+srv://...
  MFLIX_NS=sample_mflix


It's highly suggested you also change the **SECRET_KEY** to some very long, very
random string. While this application is only meant for local use during this
course, software has a strange habit of living a long time.

When you've edited the file, rename it to **.env** with the following command:

.. code-block:: sh

  mv dotenv_unix .env  # on Unix
  ren dotenv_win .env  # on Windows

*Note:* Once you rename this file to **.env**, it will no longer be visible in
Finder or File Explorer. However, it will be visible from Command Prompt or
Terminal, so if you need to edit it again, you can open it from there:

.. code-block:: sh

 vi .env       # on Unix
 notepad .env  # on Windows

In the **mflix-js** directory, run the following commands:

.. code-block:: sh

  # install MFlix dependencies
  npm install

  # start the MFlix application
  npm start

This will start the application. You can then access the MFlix application at
`http://localhost:5000/ <http://localhost:5000/>`_.


Running the Unit Tests
----------------------

To run the unit tests for this course, you will use `Jest
<https://jestjs.io/docs/en/getting-started>`_. Jest has been included in this
project's dependencies, so ``npm install`` should install everything you need.

Each course lab contains a module of unit tests that you can call individually
with ``npm test``. For example, to run the test **connection-pooling.test.js**,
run the command:

.. code-block:: sh

  npm test -t connection-pooling

Each ticket will contain the exact command to run that ticket's specific unit
tests. You can run these commands from anywhere in the **mflix-js** project.


Importing Data (Optional)
-------------------------

**Note: if you used Load Sample Dataset, you can skip this step.**

The ``mongorestore`` command necessary to import the data is located below. Copy
the command and use an Atlas SRV string to import the data (including username
and password credentials).

Replace the SRV string below with your own:

.. code-block:: sh

  # navigate to mflix-js directory
  cd mflix-js

  # import data into Atlas
  mongorestore --drop --gzip --uri \
    "mongodb+srv://m220student:m220password@<YOUR_CLUSTER_URI>" data

The entire dataset contains almost 200,000 documents, so importing this data may
take 5-10 minutes.