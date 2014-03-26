Hello Java Sample in JBoss using Simple Buildpack
==================================================

This simple Hello World servlet app illustrates the usage of the
simple bash-based JBoss buildpack.

Requirements

  * Java (to build application)
  * [Maven](http://maven.apache.org/)  (to build application)
  * Stackato instance on network
  * stackato client

Step 1: Target Stackato
-----------------------

  stackato target stackato-XXXX.local
  stackato login


Deploy Application (must be pre-built)
--------------------------------------

	stackato push -n

Run Application in Cloud
------------------------

        stackato open


## Simple Buildpack

As discussed I put together a basic buildpack to provision a
JBoss-hosted application in Stackato and configure its network
properties. This is a first step towards getting the WCG POC properly
deployed, and will likely evolve I more about the application
requirements and properties.

The buildpack is here:

   https://github.com/bcferrycoder/simple-jboss-buildpack


## Buildpack Structure

At minimum a buildpack is a git repo consisting of a bin directory
holding three scripts, of which only the compile script is used here:

 detect:   determine if the buildpack is capable of provisioning the app
 compile:  "transform" the application for the PaaS
 release:  specify entry point and other startup process

When a buildpack is explicitely requested in the application manifest
or on the command line, the detect script is ignored. Therefore the
detect script I included is basically a no-op.

The release script is similarly ignored, since the app entry point is
specified in the Procfile.

Thus in this buildpack the only script of concern is the compile
script.

Compile Script

The compile script transforms the application (i.e., prepares the appl
to run in the cloud). In our case this script installs the app
dependencies including the JRE and JBoss. It assummes that the
compiled application (.war or .ear) will be prebuilt, and thus maven
is not needed.


# Hello JBoss Application

The Hello JBoss application is deployed to Stackato using the above
buildpack. For now the base directory of this app includes:

  stackato.yml              directs the deployment process
  config/standalone.xml     a modified JBoss configuration file that specifies the network properties
  hello-jboss.war           the prebuild app, deployed to Stackato-hosted JBoss instances
  Procfile                  specifies the application entry point

## Application Manifest: stackato.yml

The manifest is loaded by Stackato at push time and provides
app-specific directives to control the deployment process.  The
interesting parts of this are:

  buildpack: the url of the buildpack, currently hosted on github 

  hooks: these specify commands to be invoked at various phases of the deployment process

  env:  specify the environment for the app, JBoss, and Java

## Configuration file:  config/standalone.xml

This file is bundled with JBoss. The original has been modified to
specify the network properties such as network interface and port to
bind to, as designated by Stackato. The modified file is included in
the application directory and copied to the Stackato container
as part of the deploy process.

## Hooks

Stackato hooks allow additional configuration at various phases of the
deployment lifecycle. Here hooks are used to update the network
properties of JBoss instance.

Three hook types can be specified:

pre-staging: invoked, once per app, at the beginning of the staging
process

post-staging: invoked, once per app, after the staging process

pre-running: invoked once per app **instance** prior to launching the app 

Here we are using the post-staging hook to copy the config/standalone.xml file into the JBoss configuration directory prior to launch.
 pre-running: 


Todo

1. This buildpack should install JBoss in version-independent
location, like /jboss. Currently JBoss ends up in
/home/stackato/jboss-as-7.1.1.Final.

Variations

Currently a modified JBoss configuration file is
(config/standalone.xml) is bundled with the application itself, and is
copied into JBoss with a staging hook.


This buildpack is extremely simple and compact, based on a single xx
line bash script. I chose this approach as I believe it's the fastest
track to getting the WCG proof of concept out the door.


In the future we might want to replace this buildpack with a variation
of the standard Cloud Foundry Java buildpack.  The above bash-
buildpack is highly tailored to your application stack, while the CF
buildpack is much more general and flexible as supports a wide range
of runtimes, languages, frameworks, and configuration options.

Each approach has its advantages and disadvantages depending on your
overall requirements and goals. I look forward to discussing these as
the buildpack evolves.

As always let me know if you have questions.








Obtain bind port/interface from environment, specify these in Stackato.yml

------------------------------------------
