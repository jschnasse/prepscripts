# Preparing Fedora for DNSCore

This document is intended not to describe the fedora installation itself, but 
the configuration needed in order to let DNSCore work with it.

(To be translated and formatted)

## Presentation Repository

The so called "Presentation Repository" stores derivates (proxies) of your AIP. Although in OAIS terms, they are DIP too, in DNSCore they are called PIP (Presentation IP). The stored files to be considered to be browser readable. The presenation repository. DNSCore uses Fedora Commons 3.5 

### Prerequsites 

1. tomcat 
2. Postgres

### Installation of Fedora Commons for DNSCore

Fedora commons 3.5 Setup:

Create Database and user for fedora.

    sudo java -jar fcrepo-installer-3.5.jar
    
### Installation

    Installation type: custom
    Authentication requirement for API-A: false
    SSL availability: false
    Servlet engine: existingTomcat
    Tomcat home directory
    Tomcat HTTP port: 8080
    Tomcat shutdown port: 8005
    Database: postgresql
    Postgresql JDBC driver: included
    Database username: fedora
    JDBC URL: jdbc:postgresql://localhost/fedora
    JDBC DriverClass: org.postgresql.Driver
    Enable FeSL AuthN: true
    Enable FeSL AuthZ: true
    Policy enforcement enabled: true
    Low Level Storage: akubra-fs
    Enable Resource Index: true
    Enable Messaging: false
    Deploy local services and demos: false

Ensure tomcat owner can access the Fedora home dir.

Open `/opt/fedora/server/config/fedora.fcfg` and change adminEmailList and repositoryName according to your needs.
     
    repositoryDomainName = danrw.de
    repositoryName = DA-NRW Presentation Repository

In the module `org.fcrepo.server.resourceIndex.ResourceIndex` change the attribute `datastore` to `localPostgresMPTTriplestore`.

Open `/opt/tomcat/webapps/fedora/WEB-INF/applicationContext.xml` and change the attribute `fedoraServerHost` to the DNS name of the machine fedora is running on.
```xml
<bean class="org.fcrepo.server.config.Parameter">
  <constructor-arg type="java.lang.String" value="fedoraServerHost">
    <!-- Defines the host name for the Fedora server, as seen from
         the  outside world. -->
  </constructor-arg>
  <property name="value" value="<hostname>" />
</bean>
```

Restart tomcat.


### Policies

In order to prevent access to non-public objects a set of XACML policies has to be installed as follows:

Remove the defaut bootstrap policies (otherwise they will be loaded at each restart):

    sudo rm -f /opt/fedora/pdp/policies/*

In order to load the custom policies for the DA-NRW check out the scripts and policies from bazaar:

    bzr checkout sftp://[login]@repositories.hki.uni-koeln.de/repositories/bzr/danrw/Fedora/trunk

The policy objects are stored in the folder `trunk/policies` and can be loaded with the script `scripts/setup-policies.py`. In order for this to work the python package `python-httplib2` has to be installed. Also the Fedora URL should be changed in the scripts before running the script with:

    python scripts/setup-policies.py

In order for the policies that restrict access to non-public objects to work the attribute finder has to be reconfigured as follows. Open the file `/opt/fedora/pdp/conf/config-attribute-finder.xml` and change the line
```xml
<attribute designator="resource" 
    name="info:fedora/fedora-system:def/model#ownerId"/>
```
to
```xml
<attribute designator="resource" 
      name="info:fedora/fedora-system:def/model#ownerId">
    <config name="target" value="object"/>
</attribute>
```

At last restart tomcat again.