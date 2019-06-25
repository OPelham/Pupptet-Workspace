# Puppet Workspace

	For learning puppet

## Online Resources

    https://cloud.google.com/community/tutorials/puppet-in-google-cloud-platform
    https://www.digitalocean.com/community/tutorials/how-to-install-puppet-4-in-a-master-agent-setup-on-ubuntu-14-04
    https://puppet.com/docs/puppet/6.4/puppet_index.html
    https://www.digitalocean.com/community/tutorials/getting-started-with-puppet-code-manifests-and-modules


## Concepts

### Puppet Master

The main server, manages all agent servers. Linux based machine

### Puppet Agent

Node Controlled by Master
checks in with master every 1800s by default.
any update needed?
if so pulls from master (rather than pushed from master)

### Standalone deployment Model

Instead of master-agent all on one machine, used for dev, test, POC

### Dataflow

master node with puppet code managment admin logs in here to make changes
secure certificates signed by master for agents (connection with agent on port 8140)

3 step process for communication
once conncted
agent sends facts (info about agents own state) eg host name, kernal, file names...
Master compiles configuration for agent based on facts - the catelog (what changes to make)
Agent use this to make changes if needed, executes configuration drift
agent then sends report back to master saying changes made (can integrate reports with 3rd party tools...)

### Manifest

Puppet programmes = manifests. made of puppet code, file names end in .pp

defualt main manifest is:

    /etc/puppet/manifests/site.pp

<!-- include a few examples -->



### Modules

Basic building blocks of puppet. Each module manages a specific task, eg installing and configuring a certain piece of software.

must be installed in puppet modulepath. Puppet loads all conteent from every module in module path

can download and install these from eg Puppet Forge, or write these yourself.

They are a collection of manifests and data (facts, templates, files)

best practice to use modules to organise almost all of your puppet manifests

To add a module place it in /etc/puppet/modules 

#### Module Structure

Specific dir structure allows puppet to find and load different things eg classes, facts, providers...

see:

    https://puppet.com/docs/puppet/6.4/modules_fundamentals.html#concept-1234
    https://puppet.com/docs/puppet/5.3/modules_fundamentals.html


### Resources

Puppet code is mainly resource declarations, describing the state of the system.

eg a resource declaration might declare a user and the attributes associated:

    user { 'mitchel':
        ensure => present,
        uid => '1000',
        gid => '1000',
        shell => '/bin/bash',
        home => '/home/mitchell'
    }

this follows the format

    resource_type { 'resource_name'
        attribute => value
        ...
    }

can list all default types available with:

    puppet reosurce --types

Resources for eg package, user

### Classes

code blocks that can be called elsewhere, make reading manifest easier

#### Class Definitions

compose a class makes it available in manifests

an eg for fomatting

    class example_class {
        ...puppet code goes here...
    }

#### Class Declaration

when class is called in manifest. Tells puppet to evaluate code within this class.

Two types:
- normal
- resource-like

**normal**

include keyword is used

    include example_class

**resource-like**

class is declared like a resource

    class { 'example_class': }

can specify class paramaters which override the class default


### Communication
agent node sends facts to master and requests catelog

master compiles and returns catelog based on sources it has access to

agent applies catelog to node, checking each resource described in catalog. If not in desired state corrects this

agent sends back report to master

Note: all communication through https with SSL certificates.
Agents automatically request certificates from master you sign these with puppetserver ca command.

### File Structure

    https://puppet.com/blog/magic-directories-guide-to-puppet-directory-structure

#### Code and Data Directory (codedir)
main dir for puppet code and data.

Used by puppet master and puppet apply (not agent)

Contains:
- environments
    - These contain your manifests and modules
- global modules directory for all environments
- Heira data and configuration

#### Config Directory (confdir)
main directory for puppet configuration

Contains:
- config files
- SSL data


#### Configuration
How to assign groups and configurations based on agent types

Classes if ancesotr node group has a class all desendents have that class too

these also inherit all class parameters and variables

Rules - node group can only marcg nodes that all ancestors also match. So specifying rules in child node group can narrow down  nodes in parent group

NOTE node can mach diff node groups causing conflicts

EG create node group webservers
add classes here that need to be applied to all web servers

then can specify subsets with differing commonalities

Once node groups created can manualy or dynamically add nodes to a group
static/dynamic

WIth rules or manually pinning nodes

#### Architecture
facts from agent to master
facter command to gather system inventory as key:value pairs
compares that received and masters config to esstablish drift between desired and running state

##### Building blocks
Resources can be grouped into classes 
Manifest end with .pp and puppet dsl (domain specific language) files definitions of puppet classes
modules collections of files def like manifests, class definitions etc

##### REsources
custom resource types or predefined
written as .pp

Resources have attribute and value (attribute => "value")

FINDINF RESOURCE TYPES

    puppet resource --type

then use:

    puppet describe [type]

to check which attributes

can use 

    puppet help [what to find]

##### excersices

create, check, test, run

###### create


###### check

    puppet parser validate [resource].pp

sill see if syntax errors present

###### test

smoke test

    puppet apply [resource].pp --noop

will show what changes would be applied

###### run

    puppet apply [resource].pp

Q: WHERE are resources, classes etc stored?

#### Main Manifest Direectory
puppet starts compiling catalog with single manifest fule or with dir of manifests which are treated as a single file. Starting point is "main manifest" or "site manifest"

#### modulepath
master service and puppet apply load most of their content from modules across one or more dirs. list of dirs for puppet to look in is "modulepath"

To view can use 

    puppet config print modulepath

#### SSL directory (ssldir)
certificates stored here. similar structure on all nodes

#### Cache direcrtory (vardir)
puppet generates data and stores it here. can mine this for analysis, or integrate other tools with puppet

### What is Heira
puppet's built in key/value data lookup system

puppet automatically searches this yaml/json for class parameters. can use to configure module

# Udemy - Puppet for Absolute Beginners

## Overview
adding user to many servers:
script sent from central server 'golden host'
issues with many diff OSs on servers
need scripts that work with each OS on for each potentially

With script same result with out worrying about OS
puppet takes care of the rest

infrastructure as code

Idempotency
stays the same regardless of number of times run.
eg no error if enable and already enabled, will just skip over it

## Setup Puppet

### Master
fqdn of puppet mastere recomended as host name (fully qualified domain name - includes all domain levels )
add to etc/hosts file so remains consitent

min 2vCPU, 1gb Ram - if under 1000 nodes

Master needs to have port 8140 open to connect with agents (https)

correct time set up on both - ntp for puppet environment 

Installing puppet on linux - package manager eg yum, apt-get, or through source with git

### Using Virtual Box

- download virtual box
- download centOS from osboxes.org
- move vdi into HOME/Library/VirtualBox/HardDisks/.
- when created -> settings -> network NAT, advanced -> port forwarding name ssh host port 2222 guest port 22
- take snapshot 
- login and password osboxes.org as default
- ifconfig -a
- sudo -i
- check ssh with systemctl status sshd
- ssh root@127.0.0.1 -p2222
- ping google.com

### Pupet self contained example

check ip address - will be 10.0.2.15
vim /etc/hosts
add line with above ip address and puppetselfcontained.example.com

suggests fqdn
and this set to be persistent

set host name as the above with 

    hostnamectl set-hostname puppetselfcontained.example.com

check this worked with 

    hostname

https://puppet.com/docs/puppet/5.5/puppet_platform.html#yum-based-systems

the above has command for enterprise linux 7

    sudo rpm -Uvh https://yum.puppet.com/puppet5-release-el-7.noarch.rpm

To check all poackages

    yum list | grep -i puppet

now install puppet

    yum install puppet-agent.x86_64

systemctl status puppet
systemctl start puppet
systemctl status puppet

Setting up ageents
- need connectivity (could ping)
- master port 8140 to connect with agents
- NTP server in environment so all are in sync for time

Connecting master and agent
puppet configuration file for each master and agent
at:
    /etc/puppetlabs/puppet/puppet.conf
COuld differ based on OS

master config file

    [main]
    certname = [here put FQDN of puppet master] puppet.example.com
    server = puppet.example.com

agent config file

    [main]
    certname = agent01.example.com
    server = puppet.example.com

then run 

    puppet agent --test
this verifies connectivity between both
But need secure connection
SSL certificates

master is certificate authority for all agents

agent first checks if has signed cert from master

if not requests one

master signs and sends back

used for all future communication between the two

tracked under /etc/puppetlabs/puppet/ssl/ in both agent and master

systemctl start puppet runs connection from agent?

on master then

    puppet cert list
will show all certificates waiting certification

    puppet cert sign [name of agent eg agent01.example.com]
to manually approve one

autsign automatically sign from predefined list of servers

need file called autosign.conf

in /etc/puppetlabs/puppet/

in this could have hostnames or wildcard patterns eg:

    agent02.example.com
    *.company.com

Then restart puppet master services after change

    systemctl restart puppetserver

other commands

    puppet cert clean
clears all existing  certs from master

    puppet cert generate
explicit make certificate




# Tutorial on how to develop a manifest

    https://www.digitalocean.com/community/tutorials/getting-started-with-puppet-code-manifests-and-modules



<!--
##### abstraction
- resources
	eg
	- file
	- package
	- service
	- user

- Custom resource types can then bee bundled into custom resource types and classes and grouped into modules

so can have webserver with out having to define all

	Node: ind server/device managed by Puppet

	Resources: Ind units of configuration in puppet

	Class: collection of puppet code that makes sense as a logical group. Put together under name that can be used by others.
	include set of resources by name	

	Manifest: text file for holding puppte code, single class or defined resource type

	Profile: Class defining specific config, gen from other classes groups

	Role: business role of node defined, sev profile classes, code duplication encouraged

-->

# Project set up
- virtual box
- vagrant


Modules: held on puppet forge, templates to save from reinventing the wheel
 

Catalogs
describe desired state
lists all resources to be managed and the dependencies between these resources

2 stages for configuration
- Compile catalog
- apply catalog

Nodes download catelog from master, gives desired state
catelog compiled with from single manifest file or directory of these

# GCP Puppet Tutorial
    https://cloud.google.com/community/tutorials/puppet-in-google-cloud-platform

## Set up instances 
create samll compute engine instance with OS Ubuntu 16.04 allow http traffic named puppet-agent
create 1 vCPU 3.75gb mem compute engine instance same OS with default firewall option named puppet-master

ssh into puppet-master install puppet server

    wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
    sudo dpkg -i puppetlabs-release-pc1-xenial.deb
    sudo apt-get update
    sudo apt-get install puppetserver

start puppet server

    sudo systemctl start puppetserver

check it is running

    sudo systemctl status puppetserver

should see "active(running)"

configure to start at boot

    sudo systemctl enable puppetserver


now ssh into puppet-agent and request certificate from puppet master

    wget https://apt.puppetlabs.com/puppetlabs-release-pc1-xenial.deb
    sudo dpkg -i puppetlabs-release-pc1-xenial.deb
    sudo apt-get update
    sudo apt-get install puppet-agent

before staring puppet agent edit /etc/hosts file to identify puppet master to use to request the certificate. at end of file specify ip address of puppet master intance

    sudo vim /etc/hosts 
add:

    10.128.0.3 puppet-master.us-central1-a.my-user-project-241423.internal puppet-master # Added by me

then run puppet agern

    sudo systemctl start puppet
    sudo systemctl enable puppet

checking systemctl status puppet before enable showed could not request certificate name or service not known. So tried reformating in /etc/hosts

changed to ip address Puppet
stoped and started puppet and staus check did not come up with error

enabled puppet with above command

## Signing certificate to establish communication between master and agent

to list all ubsigbed certificates as puppet master

    sudo /opt/puppetlabs/bin/puppet cert list

to sign all

    sudo /opt/puppetlabs/bin/puppet cert sign --all

or sign a single...

## Now write webserver module and manifest that will install apache2 and write hello world page

On the puppet-master instance, navigate to the folder /etc/puppetlabs/code/environments/production/manifests/, make a manifest file named site.pp

    sudo touch site.pp

and copy the following:

    sudo vim site.pp 

    node /agent/{
     include webserver
    }

<!-- TODO include notes what this means, what you are doing-->

Navigate to the /etc/puppetlabs/code/environments/production/modules directory 

    cd ..
    cd modules

Make a new directory by running the following command:

    sudo mkdir -p webserver/manifests

create a file in here init.pp

    cd webserver/manifests
    sudo touch init.pp

in this file add the following:

    class webserver {
        package { 'apache2' :
        ensure => present
        }
        file {'/var/www/html/index.html': # resouce type file and filename
            ensure => present, # make sure it exists
            content => "<h1>This page is installed from Puppet Master</h1>", # content od the file
    
        }

    }

Run the following command on the puppet-agent instance to get the catalog from the Puppet master and apply the manifest:

    sudo /opt/puppetlabs/bin/puppet agent --test

Copy the external IP address of the puppet-agent instance (you can get this from the VM instances page in the GCP console) and paste it into your browser. You should see a web page with the message "This page is installed from Puppet Master".
