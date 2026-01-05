# Standalone MQ console 

# Table of Contents {#table-of-contents .TOC-Heading}

[1 Introduction [7](#introduction)](#introduction)

[1.1 About this hands-on lab
[7](#about-this-hands-on-lab)](#about-this-hands-on-lab)

[1.1.1 Prerequisites [7](#prerequisites)](#prerequisites)

[1.2 Core concepts [7](#core-concepts)](#core-concepts)

[1.2.1 Remote console [7](#remote-console)](#remote-console)

[1.2.2 Local console [7](#local-console)](#local-console)

[1.2.3 Authentication of console users
[7](#authentication-of-console-users)](#authentication-of-console-users)

[1.2.4 Authorisation of console users
[8](#authorisation-of-console-users)](#authorisation-of-console-users)

[1.2.5 Queue Manager Authentication
[8](#queue-manager-authentication)](#queue-manager-authentication)

[1.2.6 Queue Manager Authorisation
[8](#queue-manager-authorisation)](#queue-manager-authorisation)

[1.3 Practical use cases with the available roles
[10](#practical-use-cases-with-the-available-roles)](#practical-use-cases-with-the-available-roles)

[1.4 Overview of the main lab tasks
[11](#overview-of-the-main-lab-tasks)](#overview-of-the-main-lab-tasks)

[2 Install IBM MQ Server
[12](#install-ibm-mq-server)](#install-ibm-mq-server)

[2.1 Completed prerequisites (commands in green do not need to be
performed) [12](#_Toc182819886)](#_Toc182819886)

[2.2 Install Ansible [13](#install-ansible)](#install-ansible)

[2.3 Install IBM MQ using Ansible
[13](#install-ibm-mq-using-ansible)](#install-ibm-mq-using-ansible)

[2.3.1 Create an MQSC commands file
[13](#create-an-mqsc-commands-file)](#create-an-mqsc-commands-file)

[2.3.2 Create an Ansible playbook
[13](#create-an-ansible-playbook)](#create-an-ansible-playbook)

[2.3.3 Run the Ansible playbook
[13](#run-the-ansible-playbook)](#run-the-ansible-playbook)

[3 Install the standalone IBM MQ web server
[14](#_Toc182819892)](#_Toc182819892)

[3.1 Create a user to run the web server
[14](#create-a-user-to-run-the-web-server)](#create-a-user-to-run-the-web-server)

[3.2 Install the web server software
[15](#install-the-web-server-software)](#install-the-web-server-software)

[3.3 Setup the environment
[16](#setup-the-user-environment)](#setup-the-user-environment)

[3.3.1 Configure MQ Console security
[17](#configure-mq-console-security)](#configure-mq-console-security)

[3.3.2 Configure the console
[18](#configure-the-console)](#configure-the-console)

[3.3.3 Login to the console
[18](#login-to-the-console)](#login-to-the-console)

[3.4 Setup access on the remote Queue Manager
[19](#setup-access-on-the-remote-queue-manager)](#setup-access-on-the-remote-queue-manager)

[3.4.1 Setup the operating system users and groups
[19](#setup-the-operating-system-users-and-groups)](#setup-the-operating-system-users-and-groups)

[3.4.2 Start the queue manager and web server
[19](#start-the-queue-manager-and-web-server)](#start-the-queue-manager-and-web-server)

[3.4.3 Setup the queue manager user access
[19](#_Toc182819902)](#_Toc182819902)

[3.5 Authorize the developers group and AdminRO role for RO access
[20](#authorize-the-developers-group-and-adminro-role-for-ro-access)](#authorize-the-developers-group-and-adminro-role-for-ro-access)

[3.6 Authorize developers group for DEV\* queues
[20](#authorize-developers-group-for-dev-queues)](#authorize-developers-group-for-dev-queues)

[4 Add a remote queue manager to the standalone console
[21](#add-a-remote-queue-manager-to-the-standalone-console)](#add-a-remote-queue-manager-to-the-standalone-console)

[4.1 Add using manual entry
[21](#add-using-manual-entry)](#add-using-manual-entry)

[5 Explore the Console [23](#explore-the-console)](#explore-the-console)

[5.1 Connect an application
[23](#connect-an-application)](#connect-an-application)

[5.2 Put a message using the console
[23](#_Toc182819909)](#_Toc182819909)

[5.3 Try to create a queue called TEST
[23](#_Toc182819910)](#_Toc182819910)

[6 Getting help and troubleshooting
[24](#_Toc182819911)](#_Toc182819911)

[6.1 Common troubleshooting tips
[24](#common-troubleshooting-tips)](#common-troubleshooting-tips)

[6.2 Getting help [24](#getting-help)](#getting-help)

[7 Appendix -- Dev MQSC configuration
[25](#appendix-dev-mqsc-configuration)](#appendix-dev-mqsc-configuration)

[8 Appendix -- Ansible Playbook YAML
[27](#appendix-ansible-playbook-yaml)](#appendix-ansible-playbook-yaml)

[8.1 Add a queue manager using a CCDT file
[28](#_Toc182819916)](#_Toc182819916)

# Introduction

The IBM MQ Console is a browser-based graphical user interface that lets
you easily visualize and manage your IBM MQ systems. With the console
you can quickly troubleshoot issues, create and test queues, see your
messages, set permissions and properties, and much more. Administrators
can use the MQ Console to administer queue managers and developers can
use it to test and debug applications.

## About this hands-on lab

In this lab we are going to setup the IBM MQ Web Server to provide an
IBM MQ Console that has the qualities and capabilities for use in a
large-scale enterprise environment. You will be deploying the IBM MQ
standalone console on one server and an IBM MQ queue manager on another
server, which you will connect to remotely from the standalone console.
Note that I will typically refer to the IBM MQ Console as the console.

### Prerequisites

To complete this lab, you will need two Linux servers, these can be bare
metal or VMs. The instructions are based on a Red Hat Enterprise Linux
(RHEL) server, but you can use any supported version of Linux. We will
call one server the 'mq-server', this will host a remote queue manager,
the other 'mq-web' which will host the standalone console.

Red Hat Ansible (installation instructions provided)

## Core concepts

Before we start there are some fundamental concepts that must be
explained. We will discuss these concepts and walk through a diagram
that shows how everything fits together before proceeding with the lab.

### Remote console

New for IBM MQ 9.4 is the ability to connect to a remote queue manager
using a client connection. This has several advantages; it decouples the
console from the queue manager allowing you to upgrade to the latest
console as soon as a new release is available, it also takes some of the
load off the queue manager host, and it allows you to manage your queue
managers from a central location.

### Local console

The traditional way of running the console was to install it alongside
the queue manager and run it on the same server in bindings mode. In
local mode the console makes an in memory (bindings) connection to the
queue manager.

### Authentication of console users

Authentication of users is performed at the console level and not the
queue manager, authorization is still performed by the queue manager.
The MQ Console can and should be configured to interact with your
enterprise authentication systems. Full details and options are in the
IBM MQ documentation, for this exercise we will use the built-in
authenticator to emulate something like an LDAP server.

### Authorisation of console users

#### Roles

Once a user has been authenticated, they are assigned a role at the
console level. There are 3 roles for the MQ Console.

**MQWebAdmin** -- full access -- remote connections use a privileged
user, authenticated by a username and password.

**MQWebAdminRO** -- full read-only access -- remote connections use the
same privileged user, but the console restricts what a user can do.

**MQWebUser** -- this is a special role. Any user with this role will
have their authenticated username passed through to the queue manager
for authorisation. The privileged user is still used for the connection
to the queue manager, but the privileged user will be substituted with
the console authenticated user for authorisation against queue manager
objects.

#### User groups

Groups can be used to simplify and ease maintenance by allowing you to
map a role to a group or user. In this exercise we will create a group
at the console level using the built-in security capability.

### Queue Manager Authentication

Connections from the web server running the IBM MQ Console application
to a remote queue manager use an IBM MQ client connection. In the lab we
will use a username/password for authentication at the queue manager.

### Queue Manager Authorisation

When the roles **MQWebAdmin** and **MQWebAdminRO** are used, the same
user that is used for remote connections will be used for authorisation
on the queue manager; that is either the user that is running the
console in bindings mode or in our case - the user specified for the MQ
client connection in remote client mode.

When you use the role **MQWebUser,** the user that was used to log into
to the IBM MQ Console is used for authorisation on the queue manager.
The console application will make use of the IBM MQ Alternate User ID
privilege to switch the user identifier in the PCF message - to that of
the user that authenticated during the console login procedure.

Figure 1 (below) explains the interaction of the users, groups and
roles.

Figure 1 overview of the security interaction between the console and
the MQ server, we will discuss this in detail.

![alt text](./images/image5.png)

## Practical use cases with the available roles

These are the use cases and roles that are applicable to the IBM MQ
Console.

Administrator

1.  Full access all objects (MQWebAdmin)

2.  Read-only all objects (MQWebAdminRO)

Developer

3.  Full read-only access plus authority to owned objects
    (MQWebAdminRO) + (MQWebUser)

4.  Access to owned objects only (MQWebUser), accept there will be
    errors on panels and then either,

    a.  Supress errors to avoid flooding error log

    b.  Leave generated errors in the log

Options 1 and 2 are likely to be reserved for administrators, and in
most cases option 1 would have a break-glass account to ensure it is
only accessed in an emergency.

Option 2 could be used by a developer in a test environment, but they
will most likely want to make use of the test change and test
capabilities available in the console. To enable this, you can allow
grant more access to specific objects by using IBM MQ authorisation at
the queue manager level -- option 3.

Option 4 is the most restrictive, but it has some drawbacks presently;
when the console is used with just the MQWebUser role it will still try
to access objects for the overview panels etc. this will generate
authorisation warning messages in the queue manager's logs. To avoid
this, you can supress the errors, but this probably isn't something you
want to do, the alternative is to leave them, but that isn't ideal
either.

The recommendation is that you use option 1 for administrator access to
all environments and use a break-glass user account for production. Use
option 2 for developers in test environments and don't give developers
access to production unless you are operating in a DevOps environment.
Add additional access, option 3, for developers to be able to manage
their objects. You will see later how we can use a wildcard to simplify
that process.

## Overview of the main lab tasks

# Install IBM MQ Server

***Time to complete \~20 minutes***

In this step we will use Red Hat Ansible to install IBM MQ server onto
the 'mq-server' machine. An automated Ansible playbook will create a
queue manager called QM1 and start a local console. We will use this
queue manager as the remote queue manager, we can also use its locally
installed console to demonstrate how to easily get a JSON CCDT file that
can be used by the standalone console to connect to a remote queue
manager.

## Install Ansible

If not already logged in as user 'root' switch to the 'root' user.

\$ sudo su -

Install Ansible and the IBM MQ galaxy collection,

\$ dnf install ansible-core -y

\$ ansible-galaxy collection install ibm_messaging.ibmmq

## Install IBM MQ using Ansible

In this step we will install IBM MQ using Ansible and a sample playbook
that can be found in the appendix, more details on this process can be
found here,

<https://community.ibm.com/community/user/integration/blogs/martin-evans/2023/12/13/a-quick-guide-to-installing-ibm-mq-on-ubuntu-linux>

Note: do not use the dev-config.mqsc file that comes with Ansible.
Instead copy and paste the dev-config.mqsc from the appendix in this
document; it has 2 extra commands, one to turn mutual TLS (MTLS) off and
another to map the 'mqweb' user to 'mqm' for administrator access.

### Create an MQSC commands file

Create a file called dev-config.mqsc, this will be applied to the queue
manager that Ansible creates.

\$ vi dev-config.mqsc

Copy the example MQSC code from the appendix and save the file.

### Create an Ansible playbook

Create a file called 'setup-playbook.yml' and copy the
setup-playbook.yml sample from this document's appendix,

\$ vi setup-playbook.yml

Paste the text from the appendix and save the file.

### Run the Ansible playbook

This command will setup users, install IBM MQ, create a queue manager
called 'QM1' and run the dev-config.mqsc file against the queue manager.

\$ ansible-playbook setup-playbook.yml -e \'ibmMqLicence=accept\'

After the playbook has finished switch to the 'mqm' user and check that
you have a running queue manager called 'QM1'

\$ su -- mqm

\$ dspmq

You should see,

QMNAME(QM1) STATUS(Running)

[]{#_Toc182819892 .anchor}

# Install the standalone IBM MQ web server

***Time to complete \~20 minutes***

In this section we will install the IBM MQ standalone web server, the
web server runs both the MQ Console application and the MQ REST API. As
of IBM MQ version 9.3.5, you can install the standalone IBM MQ web
server from a download that is available on Fix Central, for this lab we
will be using version 9.4.0.0.

## Create a user to run the web server

The user that starts the web server is the user that is used in bindings
mode for connections to local queue managers, that is queue managers on
the same host as the console. For this exercise we will be making remote
client connections that use a username/password that is authenticated by
the server running the remote queue manager, but we still need to create
a regular operating system user to run the web server.

Log into the 'mq-web' server as user 'root' or a user that can add users
to the system and create a user and password for user 'mqweb' e.g.

\$ useradd mqweb

\$ passwd mqweb

or

\$ sudo useradd mqweb

\$ sudo passwd mqweb

This will add a user called 'mqweb', set the password, and create a home
folder called '/home/mqweb'

## Download the standalone console software

Use a browser to log into fix central and download the console software
for your platform, Linux X64 in this case.

<https://ibm.biz/mq94webserver>

The download package is a tar.gz file, for example
9.4.1.0-IBM-MQ-Web-Server-LinuxX64.tar.gz

Copy the package to your 'mq-web' server using the 'mqweb' user account,
you can use something like WinSCP to copy the file.

## Install the web server software

Log in as user 'mqweb'

\$ mkdir software

Extract the tar file e.g.

\$ tar -xf 9.4.1.0-IBM-MQ-Web-Server-LinuxX64.tar.gz -C software/

## Setup the user environment

Before we can start the web server there are some configuration steps
that must be completed.

Create a folder for the web server instance,

\$ mkdir /home/mqweb/var

Modify the bash login script of the user 'mqweb' to setup the
environment variables whenever we login,

\$ vi \~/.bash_profile

Add the following highlighted lines, (in vi press 'shift' 'G' to got to
the bottom of the file, press 'o', and then copy and paste the
highlighted text below, press 'esc', press ':' , enter 'wq' , and press
enter to save)

[export MQ_OVERRIDE_DATA_PATH=/home/mqweb/var]{.mark}

[export PATH=\$PATH:/home/mqweb/software/bin]{.mark}

Exit the session and log back in again to activate the environment
variables,

\$ exit

*If switching make sure you use a '-' to activate the profile*

\$ su -- mqweb

### Finish the console setup

Create the folder structure for the web server, once only,

\$ crtmqdir -s -f

\$ mqlicense

Accept the license, when prompted press '1' to accept

Confirm successful installation by running,

\$ dspmqweb properties -a

If you see settings in the output the MQ web server has been installed,
the next step is to configure security.

### Configure MQ Console security

Before starting the console application, we need to set up an
authorisation and authentication service. We will use basic_registry
built-in service to emulate something like an enterprise LDAP service.

As user 'mqweb' copy the sample auth service,

\$ cp /home/mqweb/software/web/mq/samp/configuration/basic_registry.xml
/home/mqweb/var/web/installations/MQWEBINST/servers/mqweb/mqwebuser.xml

Modify mqwebuser.xml,

\$ vi
/home/mqweb/var/web/installations/MQWEBINST/servers/mqweb/mqwebuser.xml

Modify the roles for the MQ Console, changes highlighted.

\<enterpriseApplication id=\"com.ibm.mq.console\"\>

\<application-bnd\>

\<security-role name=\"MQWebAdmin\"\>

\<group name=\"MQWebAdminGroup\" realm=\"defaultRealm\"/\>

\</security-role\>

\<security-role name=\"MQWebAdminRO\"\>

\<user name=\"mqreader\" realm=\"defaultRealm\"/\>

[\<user name=\"fred\" realm=\"defaultRealm\"/\>]{.mark}

\</security-role\>

\<security-role name=\"MQWebUser\"\>

[\<group name=\"MQWebUserGroup\" realm=\"defaultRealm\"/\>]{.mark}

\</security-role\>

\<security-role name=\"MFTWebAdmin\"\>

\<user name=\"mftadmin\" realm=\"defaultRealm\"/\>

\</security-role\>

\<security-role name=\"MFTWebAdminRO\"\>

\<user name=\"mftreader\" realm=\"defaultRealm\"/\>

\</security-role\>

\</application-bnd\>

\</enterpriseApplication\>

Modify the basic registry,

\<basicRegistry id=\"basic\" realm=\"defaultRealm\"\>

\<!\--

This sample defines two users with unencoded passwords

and a group, these are used by the role mappings above.

\--\>

[\<user name=\"fred\" password=\"fred\"/\>]{.mark}

\<user name=\"mqadmin\" password=\"mqadmin\"/\>

\<user name=\"mqreader\" password=\"mqreader\"/\>

\<user name=\"mftadmin\" password=\"mftadmin\"/\>

\<user name=\"mftreader\" password=\"mftreader\"/\>

\<group name=\"MQWebAdminGroup\"\>

\<member name=\"mqadmin\"/\>

\</group\>

[\<group name=\"MQWebUserGroup\"\>]{.mark}

[\<member name=\"fred\"/\>]{.mark}

[\</group\>]{.mark}

\</basicRegistry\>

### Configure the console

You will observe from the 'dspmqweb properties -a' that the 'httpHost'
is set to 'localhost'. We will change this to '\*' to get the web server
to bind to all interfaces. As we only connect on the same VM (localhost)
it will work but this is limited, in a real-world scenario you would
bind it to a public interface.

\$ setmqweb properties -k httpHost -v \"\*\"

Enable the creation of remote connections in the console UI,

\$ setmqweb properties -k mqConsoleRemoteUIAdmin -v \"true\"

Start the web server

\$ strmqweb

This might take a few seconds, next, display the console status with,

\$ dspmqweb

### Login to the console

Get the URL from the terminal by running the 'dspmqweb' command again
(as above),

\$ dspmqweb

Copy the URL into your web browser e.g.

<https://mq-web.mydomain.com:9443/ibmmq/console>

Ignore any warnings about certificates as we are using the self-signed
certificate for now.

Login with user: 'mqadmin' password: 'mqadmin'

Click on '**Manage'**

You should see the '**Connect +**' button if you have successfully
enabled remote queue manage management.

Next, we will setup a queue manager and console on a different host that
we can connect to using a remote connection from the standalone console.
When we have completed that task, we will add it using the standalone
console UI.

## Setup access on the remote Queue Manager

On the mq-server VM.

There are two steps here, the operating system level and the queue
manager.

### Setup the operating system users and groups

Create an operating system group and a user to allow the standalone
console and user 'fred' access to the queue manager. On the mq-server VM
open a terminal window and as user 'root' run the following commands,

\$ sudo su --

\$ groupadd developers

\$ useradd fred -g developers

\$ useradd mqweb

Set a password for 'mqweb' to be 'change-this-password' or one of your
choice.

\$ passwd mqweb

### Start the queue manager and web server

If they are not already running, as user 'mqm'

\$ strmqm QM1

\$ strmqweb

## Authorize the 'developers' group and AdminRO role for RO access

These authorities are required for read-only users to access some of the
data in the overview panels. The user 'fred' will be in the 'developers'
operating system group, and he will have the read-only role. The level
of access is cumulative.

\$ runmqsc QM1

SET AUTHREC PROFILE(\'SYSTEM.REST.REPLY.QUEUE\') GROUP(\'developers\')
OBJTYPE(QUEUE) AUTHADD(BROWSE,GET,INQ,PUT,DSP)

SET AUTHREC PROFILE(\'SYSTEM.ADMIN.COMMAND.QUEUE\')
GROUP(\'developers\') OBJTYPE(QUEUE) AUTHADD(PUT,DSP)

SET AUTHREC group(\'developers\') OBJTYPE(QMGR) AUTHADD(DSP)

REFRESH SECURITY

**Optional,**

Adding a user/group to AdminRO alleviates the need for the below, but if
you are using the MQWebUser role alone you will need to grant these
permissions, TODO clarify/expand

 

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(QUEUE)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(TOPIC)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(CHANNEL)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(PROCESS)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(NAMELIST)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(AUTHINFO)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(CLNTCONN)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(LISTENER)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(SERVICE)
AUTHADD(DSP)

SET AUTHREC PROFILE(\'\*\*\') GROUP(\'developers\') OBJTYPE(COMMINFO)
AUTHADD(DSP)

## Authorize developers group for DEV\* queues

Authorities required for 'fred' to be able to access any queues starting
with 'DEV'.

 SET AUTHREC PROFILE(\'DEV.\*\*\') GROUP(\'developers\') OBJTYPE(QUEUE)
AUTHADD(ALLMQI)

# Add a remote queue manager to the standalone console

In this step we will add a remote queue manager using the standalone
console UI. We can do this manually, or by getting a JSON CCDT file from
the remote queue manager's console and then using it in the standalone
console.

## Add using manual entry

Using the mq-web [standalone]{.underline} VM's console, log in with
'mqadmin/mqadmin'

Navigate to the 'Manage' panel and click on the 'Connect' button, enter
the following details,

**Queue manager name:** QM1

Click on 'Manual entry'

**Channel name:** DEV.ADMIN.SVRCONN

**Host name:** *enter the hostname or IP address of the mq-server VM*

**Port:** 1414

Click the 'Add' button

Click 'Next'

**User ID:** mqweb

**Password:** change this password to what you set earlier on the remote
queue manager server

You should have something like this,

![A screenshot of a computer Description automatically
generated](./media/image6.png){width="7.0in"
height="3.7569444444444446in"}

# Explore the Console

For this part of the exercise, we will use the IBM MQ sample programs on
the remote queue manager QM1 to make some application connections and
put/get some messages. We will observe the application behaviour in the
remote console.

## Connect an application

Log into the mq-server VM and open a terminal.

Switch to the 'mqm' user

\$ su -- mqm

Run the sample put program

\$ /opt/mqm/samp/bin/amqsput DEV.QUEUE.1 QM1

Type something like 'test' and press enter.

Go to the standalone console and observe the connected applications,
Applications à Connected applications.

![A screenshot of a computer Description automatically
generated](./media/image7.png){width="6.721233595800525in"
height="1.9012379702537183in"}

View the message on the queue.

[]{#_Toc182819909 .anchor}![A screenshot of a computer Description
automatically generated](./media/image8.png){width="6.761421697287839in"
height="2.0659897200349957in"}

## Put a message using the console

Log out and log back into the console as user 'fred' with a password of
'fred'

Navigate to 'Queues'

Click on the ellipses (3 dots) of the queue 'DEV.QUEUE.1' and click
'Create message'

Browse the queue and you will see the different users that have put
messages.

![A screenshot of a computer Description automatically
generated](./media/image9.png){width="6.485059055118111in"
height="2.080022965879265in"}

[]{#_Toc182819910 .anchor}

## Try to create a queue called TEST

This should fail because user 'fred' does not have permission to create
a queue,

![A screenshot of a computer Description automatically
generated](./media/image10.png){width="5.879318678915135in"
height="3.167211286089239in"}

![A screenshot of a computer Description automatically
generated](./media/image11.png){width="5.912075678040245in"
height="1.649359142607174in"}

logout and log back in with user 'mqadmin' password 'mqadmin' and try
again, it should work this time.

[]{#_Toc182819911 .anchor}

# Getting help and troubleshooting

This section provides information about getting help with your lab and
some common troubleshooting topics.

## Common troubleshooting tips

The most important place to look for errors is in the IBM MQ error logs
and the console error log, check these logs for errors.

The Ansible install doesn't always provide a clear reason for failure,
add -v to get a more verbose output.

## Getting help

The IBM MQ queue manager error logs can be found on the mq-server VM
here,

/var/mqm/qmgrs/QM1/errors/AMQERR01.LOG

The standalone console log can be found on the mq-web VM here,

/home/mqweb/var/web/installations/MQWEBINST/servers/mqweb/logs

# Appendix -- Dev MQSC configuration

\* © Copyright IBM Corporation 2017, 2019

\*

\*

\* Licensed under the Apache License, Version 2.0 (the \"License\");

\* you may not use this file except in compliance with the License.

\* You may obtain a copy of the License at

\*

\* http://www.apache.org/licenses/LICENSE-2.0

\*

\* Unless required by applicable law or agreed to in writing, software

\* distributed under the License is distributed on an \"AS IS\" BASIS,

\* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
implied.

\* See the License for the specific language governing permissions and

\* limitations under the License.

STOP LISTENER(\'SYSTEM.DEFAULT.LISTENER.TCP\') IGNSTATE(YES)

\* Developer queues

DEFINE QLOCAL(\'DEV.QUEUE.1\') REPLACE

DEFINE QLOCAL(\'DEV.QUEUE.2\') REPLACE

DEFINE QLOCAL(\'DEV.QUEUE.3\') REPLACE

DEFINE QLOCAL(\'DEV.DEAD.LETTER.QUEUE\') REPLACE

\* Use a different dead letter queue, for undeliverable messages

ALTER QMGR DEADQ(\'DEV.DEAD.LETTER.QUEUE\')

\* Developer topics

DEFINE TOPIC(\'DEV.BASE.TOPIC\') TOPICSTR(\'dev/\') REPLACE

\* Developer connection authentication

DEFINE AUTHINFO(\'DEV.AUTHINFO\') AUTHTYPE(IDPWOS) CHCKCLNT(REQDADM)
CHCKLOCL(OPTIONAL) ADOPTCTX(YES) REPLACE

ALTER QMGR CONNAUTH(\'DEV.AUTHINFO\')

REFRESH SECURITY(\*) TYPE(CONNAUTH)

\* Developer channels (Application + Admin)

\* Developer channels (Application + Admin)

DEFINE CHANNEL(\'DEV.ADMIN.SVRCONN\') CHLTYPE(SVRCONN) REPLACE

DEFINE CHANNEL(\'DEV.APP.SVRCONN\') CHLTYPE(SVRCONN) MCAUSER(\'app\')
REPLACE

ALTER CHANNEL(DEV.ADMIN.SVRCONN) CHLTYPE(SVRCONN) TRPTYPE(TCP)
SSLCAUTH(OPTIONAL)

\* Developer channel authentication rules

SET CHLAUTH(\'\*\') TYPE(ADDRESSMAP) ADDRESS(\'\*\') USERSRC(NOACCESS)
DESCR(\'Back-stop rule - Blocks everyone\') ACTION(REPLACE)

SET CHLAUTH(\'DEV.APP.SVRCONN\') TYPE(ADDRESSMAP) ADDRESS(\'\*\')
USERSRC(CHANNEL) CHCKCLNT(REQUIRED) DESCR(\'Allows connection via APP
channel\') ACTION(REPLACE)

SET CHLAUTH(\'DEV.ADMIN.SVRCONN\') TYPE(BLOCKUSER) USERLIST(\'nobody\')
DESCR(\'Allows admins on ADMIN channel\') ACTION(REPLACE)

SET CHLAUTH(\'DEV.ADMIN.SVRCONN\') TYPE(USERMAP) CLNTUSER(\'admin\')
USERSRC(CHANNEL) DESCR(\'Allows admin user to connect via ADMIN
channel\') ACTION(REPLACE)

SET CHLAUTH(\'DEV.ADMIN.SVRCONN\') TYPE(USERMAP) CLNTUSER(\'admin\')
USERSRC(MAP) MCAUSER (\'mqm\') DESCR (\'Allow admin as MQ-admin\')
ACTION(REPLACE)

SET CHLAUTH(\'DEV.ADMIN.SVRCONN\') TYPE(USERMAP) CLNTUSER(\'mqweb\')
USERSRC(MAP) MCAUSER (\'mqm\') DESCR (\'Allow admin as MQ-admin\')
ACTION(REPLACE)

\* Developer authority records

SET AUTHREC PRINCIPAL(\'app\') OBJTYPE(QMGR) AUTHADD(CONNECT,INQ)

SET AUTHREC PROFILE(\'DEV.\*\*\') PRINCIPAL(\'app\') OBJTYPE(QUEUE)
AUTHADD(BROWSE,GET,INQ,PUT)

SET AUTHREC PROFILE(\'DEV.\*\*\') PRINCIPAL(\'app\') OBJTYPE(TOPIC)
AUTHADD(PUB,SUB)

\* Developer listener

DEFINE LISTENER(\'DEV.LISTENER.TCP\') TRPTYPE(TCP) PORT(1414)
CONTROL(QMGR) REPLACE

START LISTENER(\'DEV.LISTENER.TCP\') IGNSTATE(YES)

STOP LISTENER(\'SYSTEM.DEFAULT.LISTENER.TCP\') IGNSTATE(YES)

# Appendix -- Ansible Playbook YAML

\-\--

\- name: prepares MQ server

hosts: localhost

connection: local

become: true

environment:

PATH: /opt/mqm/bin:{{ ansible_env.PATH }}

collections:

\- ibm_messaging.ibmmq

vars:

version: 940

tasks:

\- name: Import downloadmq role

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.downloadmq

\- name: Import setupusers role

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.setupusers

\- name: Import installmq role

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.installmq

\- name: Import setupenvironment role

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.setupenvironment

\- name: Get MQSC file

become: true

become_user: mqm

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.getconfig

vars:

mqsc_local: dev-config.mqsc

\- name: Set up web console

become: true

become_user: mqm

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.setupconsole

\- name: Start web console

become: true

become_user: mqm

ansible.builtin.import_role:

name: ibm_messaging.ibmmq.startconsole

\- name: Create a queue manager

become_user: mqm

tags: \[\"queue\"\]

ibm_messaging.ibmmq.queue_manager:

qmname: QM1

state: present

\- name: Use our MQSC File

become: true

become_user: mqm

ibm_messaging.ibmmq.queue_manager:

qmname: QM1

state: running

mqsc_file: /var/mqm/dev-config.mqsc

[]{#_Toc182819916 .anchor}

# Add a queue manager using a CCDT file 

This approach can be used as an alternative to manually adding.

Access the mq-server VM console.

**Login**: mqadmin/mqadmin

Navigate to local queue managers and **QM1**, click on the QM1 **local
queue manager**, click '**view configuration'** and then '**actions'**
select '**Download the connection file**' from the drop-down menu.

Select the channel, DEV.ADMIN.SVRCONN

Click Next, take defaults and download the file.

Now head back over to the standalone console and use the CCDT you just
downloaded to add the remote queue manager.

Click 'Manage'

Click 'Connect +'

**Queue manager name:** QM1

Click 'Browse' and upload the CCDT you downloaded from the mq-server VM
console.

Click 'Next'

Enter user, 'mqweb', password, 'change-this-password'
