h1. Overview

Example for the "Karaf Tutorial 9 - CDI meet OSGi" 
that implements a very small application to manage a list of tasks or to dos like in tutorial 1. 
Originally I planned to use pax cdi and deltaspike jpa for this tutorial. 
Unfortunately deltaspike jpa does not yet work in OSGi. So I created a maven plugin that creates blueprint from CDI annotations.
See https://github.com/cschneider/blueprint-maven-plugin

This allows to build an example with JPA persistence, Transactions and a Servlet Ui using zero hand written blueprint xml.

It shows how to:

- Create bundles using maven and the maven bundle plugin
- Wire bundles using CDI annotations and OSGi services
- Write JPA DAO classes like in JEE using @PersistenceUnit and @Transactional
- Use the whiteboard pattern and the pax-web whiteboard extender to publish Servlets

h1. Structure

Module             | Description  
model              | Service interface and model classes shared between persistence and ui
persistence        | Full persistence implementation using JPA and hibernate
persistence-simple | Simple persistence implementation using an OSGi service and a in memory map
ui                 | Simple servlet based UI that connects to the persistence layer using an OSGi service reference and publishes a servlet 

h1. Build blueprint-maven-plugin 1.0.0

As there is no public release in central you have to build the plugin yourself

git clone https://github.com/cschneider/blueprint-maven-plugin
cd blueprint-maven-plugin
git checkout blueprint-maven-plugin-1.0.0
mvn clean install

h1. Build example

mvn clean install

h1. Installation

Download and start Karaf 3

Copy data source config to etc/org.ops4j.datasource-tasklist.cfg

Start karaf and execute the commands below

# Create data source
feature:repo-add mvn:org.ops4j.pax.jdbc/pax-jdbc-features/0.4.0/xml/features
feature:install pax-jdbc-derby pax-jdbc-config pax-jdbc-pool-dbcp2
cat https://raw.githubusercontent.com/cschneider/Karaf-Tutorial/master/tasklist-cdi/org.ops4j.datasource-tasklist.cfg | tac -f etc/org.ops4j.datasource-tasklist.cfg
service:list DataSource

feature:repo-add mvn:net.lr.tasklist.cdi/tasklist-features/1.0.0-SNAPSHOT/xml
feature:install example-tasklist-cdi-persistence example-tasklist-cdi-ui

h1. Test

Open the url below in your browser.
http://localhost:8181/tasklist

h1. Beware of bug in pax-jdbc-pool

I am using pax-jdbc-pool-dbcp2 and pax-jdbc-config to create a XA ready DataSource.

In pax-jdbc 0.4.0 there is a bug that causes the jpa code to not really work transactionally. For the example it does not matter
but for any enterprise deployment this is a severe problem. So make sure you do not deploy the version from pax-jdbc 0.4.0 to
production. pax-jdbc 0.5.0 will fix this.
See https://ops4j1.jira.com/browse/PAXJDBC-52
 