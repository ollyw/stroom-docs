# Running Stroom in an IDE (IntelliJ)

Here at Stroom Towers we tend to use IntelliJ as our Java IDE of choice. This is a guide for running Stroom in IntelliJ for the purposes of developing/debugging Stroom.

## Prerequisites

In order to build/run/debug Stroom you will need the following:

 * Java 8 JDK
 * Git
 * Maven
 * IntelliJ
 * A MySQL database server v5.6 (either installed directly or inside a Docker container)

### Clone and build _event-logging_

Stroom is dependant on the event-logging project as this is used internally for generating audit events for activity in Stroom. Until event-logging is released to a public Maven repository you must first clone and build event-logging.

```bash
git clone https://<HOST>/scm/stroom/event-logging.git
cd event-logging
mvn clean install
```

## Environment variables

Stroom relies on the environment variable _STROOM_TMP_ to tell it which directory to use for its temporary files.  In development this is also the location used by the stream store. In the absence of _STROOM_TMP_ Stroom falls back on the Java system property _java.io.tmpdir_.

You are recommended to set _STROOM_TMP_ in your _.bashrc_ / _.zshrc_ file by adding like the following:

```bash
export STROOM_TMP=/home/developer/tmp/stroom/
```

## Database setup

Stroom requires a MySQL database to run. You can either point stroom at a MySQL server or at a MySQL Docker container

### MySQL in a Docker container

To configure the stroom database in a MySQL Docker container see [Running Stroom using Docker](./docker.md) for how to set this up.

### Host based MySQL server

With an instance of MySQL server 5.6 running on your local machine do the following to create the _stroom_ database:

```bash
# log into your MySQL server using your root credentials
mysql --user=root --password=myrootpassword
```

Then run the following commands in the MySQL shell:

```sql
drop database stroom;
create database stroom;
grant all privileges on stroom.* to stroomuser@localhost identified by 'stroompassword1';
quit;
```

## Local configuration file

Stroom comes with a default configuration file but it is wise to set up your own local file to allow you to run with your own environment specific settings.

The default property file can be found in 

```
stroom-config/src/main/resources/stroom.properties
```

Create the your local configuration file and directory:

```bash
mkdir -p ~/.stroom.conf.d/
touch stroom.conf
```

Add any properties from stroom.properties that you want different values for, e.g.

```properties 
#Uncomment this to enable browser's right click menu for development
#stroom.ui.oncontextmenu=

#Connect to MySQL in Docker
stroom.jdbcDriverUrl=jdbc:mysql://172.17.0.2/stroom?useUnicode=yes&characterEncoding=UTF-8
stroom.jdbcDriverUsername=myOtherUser
stroom.jdbcDriverPassword=myOtherPassword
```

## Verify the Maven build

Before trying to run Stroom in an IDE it is worth performing a Maven build without all the tests to verify the code is in a sound state.

```bash 
mvn -Pgwt-dev-chrome -Dskip.surefire.tests=true clean install -U
```

## Sample Data

Some of the tests are dependant on some sample data and content being present in the database.  This sample data/content is also useful for manually testing the application in development. The sample data/content is generated by a class called _SetupSampleData.java_. This class assumes that the database being used for Stroom is completely empty.

First you need to create a run configuration for _SetupSampleData.java_

1. Click _Run_ -> _Edit Configurations..._
1. Click the green _+_ icon to add a new configuration
1. Select _Application_ as the configuration type
1. In the _Main Class_ field enter _SetupSampleData_
1. In the _Use classpath of module_ field select _stroom-integrationtest_
1. If you have set the _STROOM_TMP_ environment variable in your _.bashrc_ / _.zshrc_ then ignore this step.  In the _Environment variables_ field click the _..._ icon and add _STROOM_TMP_ = _~/tmp/stroom/_ (or whatever directory you choose)
1. Click the _OK_ button

Now run _SetupSampleData_ 

1. Click _Run_ -> _Run..._
1. Select _SetupSampleData_

You should now have a database populated with tables and data, providing you with some predefined feeds, data, translations, pipelines, dashboards, etc.

## Running Stroom from the IDE

The user interface for Stroom is built using GWT (see [GWT Project](http://www.gwtproject.org/) for more information or GWT specific documentation). As a result Stroom needs to be started up with GWT _Dev Mode_. _Dev Mode_ handles the compilation of the Java user interface source into JavaScript and the source map that links client JavaScript back to Java source for client side debugging.

Stroom is run from the main method in Startup.java. Before running this you need to setup the run configuration:

1. Click _Run_ -> _Edit Configurations..._
1. Click the green _+_ icon to add a new configuration
1. Select _Application_ as the configuration type
1. In the _Main class_ field enter _Startup_
1. In the _Programme arguments_ field enter 

  `-startupUrl stroom.jsp -logLevel INFO -war . -logdir . -gen . -extra . -workDir . stroom.app.AppSuperDevMode`

1. In the _Use classpath of module_ field select _stroom-startup_
1. If you have set the _STROOM_TMP_ environment variable in your _.bashrc_ / _.zshrc_ then ignore this step.  In the _Environment variables_ field click the _..._ icon and add _STROOM_TMP_ = _~/tmp/stroom/_ (or whatever directory you choose)
1. Click the _OK_ button

Now run _Startup_ 

1. Click _Run_ -> _Run..._
1. Select _Startup_

You should eventually see the GWT Dev Mode window appear.

![GWT Dev Mode Window](./resources/devMode.png)

Initially, some of the buttons shown above will not be visible as it is in the process of starting up. As soon as the _Launch Default Browser_ button appears you are ready to open Stroom in a browser. You have two options:

* Click the _Launch Default Browser_ button
* Open your preferred browser and enter the URL [http://127.0.0.1:8888](http://127.0.0.1:8888)

> **NOTE:** Stroom has been written with Google's Chrome browser in mind so has only been tested on Chrome. Behaviour in other browsers may vary. We would like to improve cross-browser support so please let us know about any browser incompatibilities that you find.

In the browser you will initially see the following:

>Starting Stroom

>Initialising context...

Once the context has been initialised you will see the Stroom blue background and a spinner while GWT compiles the front-end code. Once the code has been compiled you will be presented with the Stroom login page. Stroom in development mode uses simple username/password authentication. Enter the following credentials:

* **Username**: admin
* **Password**: admin

## Right click behaviour
Stroom overrides the defualt right click behaviour in the browser with its own context menu. For UI development it is often required to have access to the browser's context menu for example to inspect elements. To enable the browser's context menu you need to uncomment the following property in your _stroom.conf_ file, e.g.

```properties 
#Uncomment this to enable browser's right click menu for development
stroom.ui.oncontextmenu=
```
