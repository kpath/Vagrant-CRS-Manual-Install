# ATG CRS Install Guide

### Product versions used in this guide:

- Oracle Linux Server release 6.5 (Operating System)
- Oracle Database 12.1.0.2.0 Enterprise Edition
- Oracle ATG Web Commerce 11.1
- JDK 1.7
- ojdbc7.jar driver
- Jboss EAP 6.1

### About

This document describes a quick and easy way to install and play with ATG CRS.  By following this guide, you'll be able to focus on learning about ATG CRS, without debugging common gotchas.  This project contains a vagrant definition for a database vm.  If you follow the instructions in this guide you'll have a "db" vm running Oracle 12.1.0.2.0 Enterprise Edition, with a system username/password combo of system/oracle.  This is optional.  If you have another db you'd like to use, that's fine.  Just make sure it's Oracle 12c.  The benefit to running your oracle db in a VM is that you've got complete control over it, and you can set up and tear it down at will without worrying that you'll accidentally delete other peoples' data.  Plus you know it's guarateed to work because you created it with a fresh install from scratch.

You will need to refer to the [ATG CRS Installation and Configuration Guide](http://docs.oracle.com/cd/E52191_01/CRS.11-1/ATGCRSInstall/html) frequently after initial environment setup.  A lot of what this doc does is help you through the process of meeting the [installation requirements](http://docs.oracle.com/cd/E52191_01/CRS.11-1/ATGCRSInstall/html/s0203installationrequirements01.html).

### Technical Requirements

- A machine with at least 12 and preferably 16 gigs of RAM
- A minimum of 50 gigs of storage space

### Conventions

Throughout this document, the top-level directory that you checked out from git will be referred to as `{ATG-CRS}`

## Install Required Virtual Machine Software

Install the latest versions of [VirtualBox](https://www.virtualbox.org/wiki/Downloads) and [Vagrant](http://www.vagrantup.com/downloads.html)

## Download Required Database Software (If you're not using a pre-existing DB)

The first step is to download the required installers.

### Oracle 12c (12.1.0.2.0) Enterprise Edition

- Go to [Oracle Database Software Downloads](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index-092322.html)
- Accept the license agreement
- Under the section "(12.1.0.2.0) - Enterprise Edition" download parts 1 and 2 for Linux x86-64

**IMPORTANT:** Put the zip files parts 1 and 2, in the `{ATG-CRS}/software` directory at the top level of this project (it's the directory that has a `readme.txt` file telling you how to use the directory).

### Oracle SQL Developer

You will also need a way to connect to the database.  I recommend [Oracle SQL Developer](http://www.oracle.com/technetwork/developer-tools/sql-developer/downloads/index.html).

## Build the database vm (If you you're not using an external DB)

In the root directory of this project (the one that contains the Vagrantfile), type:

` vagrant up db12c`

This will set in motion an amazing series of events and can take a long time, depending on download and machine speeds:

- download an empty centos machine
- switch it to Oracle Linux (an officially supported platform for Oracle 12c and ATG 11.1)
- install all prerequisites for Oracle 12.1.0.2.0
- install and configure the oracle db software
- create an empty db name `orcl`

To get a shell on the db vm, type

` vagrant ssh db12c`

You'll be logged in as the user "vagrant".  This user has sudo privileges (meaning you can run `somecommand` as root by typing ` sudo somecommand`). To su to root (get a root shell), type `su -`.  The root password is "vagrant".  If you want to su to the oracle user, the easiest thing to do is to su to root and then type `su - oracle`.  The "oracle" user is the user that's running oracle and owns all the oracle directories.  The project directory (the directory from which you ran `vagrant up db`) will be mounted at /vagrant.  You can copy files back and forth between your host machine and the VM using that directory.

Key Information:

- The db vm has the private IP 192.168.60.4.  This is defined at the top of the Vagrantfile.
- The system username password combo is system/oracle
- The SID (database name) is orcl
- It's running on the default port 1521
- You can control the oracle server with a service: "sudo service dbora stop|start"

## Create DB schemas (Do this even if you're using your own DB, and not the one created in the steps above)

Before you can set up ATG to connect to your database, you must create four new schemas.  Just make the password the same as the user name:

- crs_core
- crs_pub
- crs_cata
- crs_catb

## Download required ATG server software

### ATG 11.1

- Go to [Oracle Edelivery](http://edelivery.oracle.com)
- Sign in or create account
- Accept the restrictions
- On the search page Select the following options: 
  - Product Pack -> ATG Web Commerce
  - Platform -> Linux x86-64
- Click Go
- Click the top search result "Oracle Commerce (11.1.0), Linux"
- Download the following parts:
  - Oracle Commerce Platform 11.1 for UNIX
  - Oracle Commerce Reference Store 11.1 for UNIX
  - Oracle Commerce MDEX Engine 6.5.1 for Linux
  - Oracle Commerce Content Acquisition System 11.1 for Linux
  - Oracle Commerce Experience Manager Tools and Frameworks 11.1 for Linux
  - Oracle Commerce Guided Search Platform Services 11.1 for Linux

### JDK 1.7

- Go to the [Oracle JDK 7 Downloads Page](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)
- Download "jdk-7u67-linux-x64.rpm"

### JBoss EAP 6.1

- Go to the [JBoss product downloads page](http://www.jboss.org/products/eap/download/)
- Click "View older downloads"
- Click on the zip downloader for 6.1.0.GA

### OJDBC Driver

- Go to the [Oracle 12c driver downloads page](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-drivers-12c-download-1958347.html)
- Download ojdbc7.jar

**IMPORTANT:** Move everything you downloaded to the `{ATG-CRS}/software` directory at the top level of this project.

## Create the "atg" vm

` vagrant up atg`

When it's done you'll have a vm created that is all ready to install and run ATG CRS.  It will have installed jdk7 at /usr/java/jdk1.7.0_67.  You'll also have the required environment variables set in the .bash_profile of the "vagrant" user.

To get a shell on the atg vm, type

` vagrant ssh atg`

Key Information:

- The atg vm has the private IP 192.168.60.5.  This is defined at the top of the Vagrantfile.
- java is installed in `/usr/java/jdk1.7.0_67`
- Your project directory is mounted at `/vagrant`.  You'll find the installers you downloaded at `/vagrant/software` from within the atg vm

## Install JBoss

Installing JBoss just involves unzipping it.  For convenience, we'll create a link to it at /home/vagrant/jboss:

```
vagrant ssh atg
unzip -n /vagrant/software/jboss-eap-6.1.0.zip -d /home/vagrant
ln -s /home/vagrant/jboss-eap-6.1 /home/vagrant/jboss
```

##  Install Endeca (Oracle Commerce Guided Search Platform)

[Documentation](http://www.oracle.com/technetwork/indexes/documentation/endecaguidedsearch-1552767.html)

ssh into the atg vagrant vm. From the directory that contains this project's Vagrantfile type:

`vagrant ssh atg`

change to the directory where your installers are:

`cd /vagrant/software`

Follow the [Getting Started Guide](http://docs.oracle.com/cd/E55323_01/Common.111/pdf/GettingStarted.pdf). **Accept all the default values.**

### MDEX

`/vagrant/software/OCmdex6.5.1-Linux64_829811.sh --target /usr/local`

Accept all defaults and then perform the post-installation procedure

`source /usr/local/endeca/MDEX/6.5.1/mdex_setup_sh.ini`

### Platform services

`/vagrant/software/OCplatformservices11.1.0-Linux64.bin --target /usr/local/`

`source /usr/local/endeca/PlatformServices/workspace/setup/installer_sh.ini`

### Tools and frameworks

You have to unzip the file V46389-01.zip into your /vagrant/software directory.  This will create the folder "/vagrant/software/cd".  You can't run the basic installer because it wants a display to be able to show you all their fancy dialogs and whatnot.  Just run the installer silently using the provided response file.  This is the same thing as accepting all the defaults.

`export ENDECA_TOOLS_ROOT=/usr/local/endeca/ToolsAndFrameworks/11.1.0`

`export ENDECA_TOOLS_CONF=/usr/local/endeca/ToolsAndFrameworks/11.1.0/server/workspace`

`/vagrant/software/cd/Disk1/install/silent_install.sh /vagrant/scripts/atg/endeca_toolsandframeworks_silent_response.rsp ToolsAndFrameworks /usr/local/endeca/ToolsAndFrameworks admin`

`sudo /home/vagrant/oraInventory/orainstRoot.sh`

### CAS
`/vagrant/software/OCcas11.1.0-Linux64.sh --target /usr/local`

## Start the services:

### Platform
`/usr/local/endeca/PlatformServices/11.1.0/tools/server/bin/startup.sh`

Logs are written to `/usr/local/endeca/PlatformServices/workspace/logs`

### Tools
`/usr/local/endeca/ToolsAndFrameworks/11.1.0/server/bin/startup.sh`

Logs are written to `/usr/local/endeca/ToolsAndFrameworks/11.1.0/server/workspace/logs`

### CAS
`/usr/local/endeca/CAS/11.1.0/bin/cas-service.sh &`

Logs are written to `/usr/local/endeca/CAS/workspace/logs`

## Install ATG

### Install ATG Platform

[Docs](http://docs.oracle.com/cd/E52191_01/CRS.11-1/ATGCRSInstall/html/s0205installingtheoraclecommerceplatf02.html)

`/vagrant/software/OCPlatform11.1.bin`

Use the following locations:

- JBoss: /home/vagrant/jboss
- Java: /usr/java/jdk1.7.0_67

### Install ATG CRS 

[Docs](http://docs.oracle.com/cd/E52191_01/CRS.11-1/ATGCRSInstall/html/s0205installingcommercereferencestore01.html)

`/vagrant/software/OCReferenceStore11.1.bin`

### Configure ATG: 

[Docs](http://docs.oracle.com/cd/E52191_01/CRS.11-1/ATGCRSInstall/html/s0208configuringtheoraclecommerceplat01.html)

Follow the CRS instructions, except as noted here. Begin with the "Starting CIM" section of the CRS setup guide.  

Exceptions to default values:

- If you've installed your database using these instructions, its IP address will be 192.168.60.4, port 1521, SID orcl
- **Very Important:** The jps-config.xml that you specify for the EAC application **MUST** be the one from the ToolsAndFrameworks install and **NOT** the one from the ATG install.  The ATG install one will give you errors when you try to create the app.  This is the correct jps-config.xml file: `/usr/local/endeca/ToolsAndFrameworks/11.1.0/deployment_template/lib/../../server/workspace/credential_store/jps-config.xml`

You can skip the "Reducing Logging Messages" and "Additional Application Server Configuration" steps.  Continue through "Accessing the Storefront"

## Gotchas

There are a couple of settings that are absolutely critical to getting this cluster running, but which aren't obvious and are difficult to diagnose if things go wrong.  Don't worry, both of these should be taken care of automatically in the scripts.

- You must have -Duser.timezone=UTC in your JAVA_OPTS variable that's used to run jboss and cim.  For some reason it's required when connecting from the atg vm to the db vm.  If you don't have this you'll get user timezone errors and ultimately can't use jdbc to connect to your database.  The vagrant setup scripts set this option for you in the vagrant user's `.bash_profile` file.
- You must use the jps-config.xml file that comes with the ToolsAndFrameworks' deployment template: `/usr/local/endeca/ToolsAndFrameworks/11.1.0/deployment_template/lib/../../server/workspace/credential_store/jps-config.xml`  If you're not sure if you set it right when configuring your application in cim, check in the file `/usr/local/endeca/Apps/CRS/config/script/WorkbenchConfig.xml`.  If you don't use the correct jps-config.xml, you'll get a variety of strange and unhelpful errors, like a ClassNotFoundException when removing your application with `runcommand.sh`



