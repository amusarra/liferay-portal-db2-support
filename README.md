![travis ci](https://travis-ci.org/amusarra/liferay-portal-db2-support.svg?branch=master)

# Welcome to DB2 support for Liferay CE 7.0 GA1/GA2

[![Join the chat at https://gitter.im/amusarra/liferay-portal-db2-support](https://badges.gitter.im/amusarra/liferay-portal-db2-support.svg)](https://gitter.im/amusarra/liferay-portal-db2-support?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Those who follow Liferay is aware of the fact that the Community Edition version 7 of Liferay, were eliminated quite a bit of components App Server, Database & Clustering Support. For more detail information you can read the blog post by [Bryan Cheung]( https://www.liferay.com/it/web/bryan.cheung/blog/-/blogs/liferay-portal-7-ce-app-server-database-clustering-support) published on April 7, 2016.

The Liferay 7 CE no more support OOTB (Out Of The Box):
* Application Server: Oracle WebLogic, IBM WebSphere
* Clustering
* MultiVM Cache
* Oracle Database, Microsoft SQL Server, IBM DB2, Sybase DB

This sample project demonstrates how to add support to the IBM DB2 database. Liferay has performed refactorting the code so that it is possible and easy to add support for databases no longer supported OOTB.

## 1. Introduction
To extend support to other databases, Liferay has decided to refactory code to use Java [*SPI (Service Provider Interface)*](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html). SPI is the mechanism that allows you to extend / change the behavior within a system without changing the source. It includes interfaces, classes or methods that the user extends or implements in order to obtain a certain functionality.

In short we must:
* Implement the SPI interface [com.liferay.portal.kernel.dao.db.DBFactory](https://github.com/liferay/liferay-portal/blob/2960360870ae69360861a720136e082a06c5548f/portal-kernel/src/com/liferay/portal/kernel/dao/db/DBFactory.java). Implementation class inside this project is **DB2DBFactory.java**
* Implement the abstract class [com.liferay.portal.dao.db.BaseDB](https://github.com/liferay/liferay-portal/blob/master/portal-impl/src/com/liferay/portal/dao/db/BaseDB.java) for DB2 DB. Implementation class inside this project is **DB2DB.java**

The following code shows how service providers are loaded via SPI.
```
public DBManagerImpl() {
  ServiceLoader<DBFactory> serviceLoader = ServiceLoader.load(
    DBFactory.class, DBManagerImpl.class.getClassLoader());

  for (DBFactory dbFactory : serviceLoader) {
    _dbFactories.put(dbFactory.getDBType(), dbFactory);
  }
}
```
To register your service provider, you create a provider configuration file, which is stored in the **META-INF/services** directory of the service provider's JAR file. The name of the configuration file is the fully qualified class name of the service provider, in which each component of the name is separated by a period (.), and nested classes are separated by a dollar sign ($).

The provider configuration file contains the fully qualified class names (FQDN) of your service providers, one name per line. The file must be UTF-8 encoded. Additionally, you can include comments in the file by beginning the comment line with the number sign (#).

Our file is called com.liferay.portal.kernel.dao.db.DBFactory and contain the FQDN of the class [it.dontesta.labs.liferay.portal.dao.db.DB2DBFactory](https://github.com/amusarra/liferay-portal-db2-support/blob/master/src/main/java/it/dontesta/labs/liferay/portal/dao/db/DB2DBFactory.java)


In the figure below shows the complete class diagram for DB2.

![Class Diagram for DB2](https://www.dontesta.it/wp-content/uploads/2014/02/DB2-1.png)

## 2. Build project
Requirements for build project

1. Sun/Oracle JDK 1.8
2. Maven 3.x (for build project) or Gradle 2.x

The driver that adds support for DB2 database is a jar (**liferay-portal-db2-support-${version}.jar**) which then will be installed in ROOT/WEB-INF/lib (for apache tomcat).

To generate the driver for IBM DB2 database just follow the instructions below.

You can download the binary jar [liferay-portal-db2-support-1.0-SNAPSHOT.jar](https://github.com/amusarra/liferay-portal-db2-support/releases/download/v1.0/liferay-portal-db2-support-1.0-SNAPSHOT.jar), by doing so you can avoid doing the build.

```
$ git clone https://github.com/amusarra/liferay-portal-db2-support.git
$ mvn package
```

the build process create the jar inside the (maven) target directory:

```
liferay-portal-db2-support-1.0-SNAPSHOT.jar
```

If you have a Gradle build system, then you can build jar by the following command

```
$ git clone https://github.com/amusarra/liferay-portal-db2-support.git
$ gradle build
```

the build process create the jar inside the build/libs directory.

## 3. Install Liferay CE 7 on DB2 Database

To install Liferay on DB2 you must have previously configured a schema for Liferay on an DB2 Database.

I have used [Virtual Appliance](https://www-01.ibm.com/marketing/iwm/iwm/web/reg/pick.do?source=swg-db2va) with a DB2 Express-C 10.1 (FP2) installation on SUSE Linux Enterprise Server

The parameters of my DB2 instance are:
* Username: db2inst1
* Password: system
* FQDN: db2.vm.local (IP: 192.168.56.101)
* TCP/IP Port: 50001

For the installation of Liferay follow the following steps:

1. Download [Liferay CE 7 GA2 Tomcat Bundle](https://sourceforge.net/projects/lportal/files/Liferay%20Portal/7.0.1%20GA2/liferay-ce-portal-tomcat-7.0-ga2-20160610113014153.zip/download) from sourceforge
2. Extract the Liferay bundle (in my case $LIFERAY_HOME is /opt/liferay-ce-portal-7.0-ga2-blog)
3. Copy the jar **liferay-portal-db2-support-${version}.jar** in $LIFERAY_HOME/$TOMCAT_HOME/webapps/ROOT/WEB-INF/lib
4. Download and install [DB2 JDBC driver (xxxxxx.jar)](http://www-01.ibm.com/support/docview.wss?uid=swg21363866) in $LIFERAY_HOME/$TOMCAT_HOME/lib/ext.
5. Create the **portal-ext.properties** in $LIFERAY_HOME with the content as the file below. ***You should modify the JDBC connection parameters to the your db and the value of liferay.home***
6. Launch the Liferay Portal through the command $LIFERAY_HOME/$TOMCAT_HOME/bin/startup.sh
7. See the Liferay activities via the log file $LIFERAY_HOME/$TOMCAT_HOME/logs/catalina.out

Below you can see the portal-ext.properties

```
##
## Admin Portlet
##
    #
    # Configure email notification settings.
    #
    admin.email.from.name=Joe Bloggs
    admin.email.from.address=test@liferay.com

##
## JDBC
##
    #
    # DB2
    #
	jdbc.default.driverClassName=com.ibm.db2.jcc.DB2Driver
    jdbc.default.url=jdbc:db2://db2.vm.local:50001/lportal:deferPrepares=false;fullyMaterializeInputStreams=true;fullyMaterializeLobData=true;progresssiveLocators=2;progressiveStreaming=2;
    jdbc.default.username=db2inst1
	jdbc.default.password=system
	
##
## Liferay Home
##
    #
    # Specify the Liferay home directory.
    #
    liferay.home=/opt/liferay-ce-portal-7.0-ga2-blog

##
## Setup Wizard
##
    #
    # Set this property to true if the Setup Wizard should be displayed the
    # first time the portal is started.
    #
    setup.wizard.enabled=false
```

You can see the part of the catalina.out log file and the contents of the DB2 Liferay database.

![Liferay CE 7 GA2 on IBM DB2 Database ](https://www.dontesta.it/wp-content/uploads/2016/08/Liferay7CEOnIBMDB2DataBase.png)

![Liferay CE 7 GA2 on IBM DB2 Database ](https://www.dontesta.it/wp-content/uploads/2016/08/Liferay7CEOnIBMDB2DataBase_1.png)
