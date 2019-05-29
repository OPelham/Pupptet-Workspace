#Puppet Workspace

	For learning puppet

### Configuration managment tool
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

	Manifest: text file for holding puppte code, single class or defined resource type

	Profile: Class defining specific config, gen from other classes groups

	Role: business role of node defined, sev profile classes, code duplication encouraged

# Project set up
- virtual box
- vagrant

