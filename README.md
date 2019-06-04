# Puppet Workspace

	For learning puppet

## Online Resources

    https://cloud.google.com/community/tutorials/puppet-in-google-cloud-platform
    https://www.digitalocean.com/community/tutorials/how-to-install-puppet-4-in-a-master-agent-setup-on-ubuntu-14-04
    https://puppet.com/docs/puppet/6.4/puppet_index.html
    https://www.digitalocean.com/community/tutorials/getting-started-with-puppet-code-manifests-and-modules


## Concepts

### Puppet Master

The main server, manages all agent servers.

### Puppet Agent



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
