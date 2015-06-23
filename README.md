# onos-for-osx
Setup instructions for developing / running ONOS on OSX

# Prereqs
Java 8, Maven, Git, Bash and Karaf

       $ mkdir ~/Applications
       $ export KARAF_VERSION=3.0.3; wget http://download.nextag.com/apache/karaf/$KARAF_VERSION/apache-karaf-$KARAF_VERSION.tar.gz -O ~/Downloads/apache-karaf-$KARAF_VERSION.tar.gz; tar -zxvf ~/Downloads/apache-karaf-$KARAF_VERSION.tar.gz -C ~/Applications/
       $ export MAVEN_VERSION=3.3.1; wget http://archive.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz -O ~/Downloads/apache-maven-$MAVEN_VERSION-bin.tar.gz; tar -zxvf ~/Downloads/apache-maven-$MAVEN_VERSION-bin.tar.gz -C ~/Applications/

Make sure you have the latest version of Java 8 installed, this can be verfied by running 

       $ /usr/libexec/java_home

# Dev env setup
The latest code can be checked out from git

       $ cd; git clone https://gerrit.onosproject.org/onos


Ensure the following is setup in your bash profile 

       $ export ONOS_ROOT=~/onos; source $ONOS_ROOT/tools/dev/bash_profile

Note the following errors are written to the console, but the env setup seems to work regardless.

       cell:unset:7: not enough arguments
       /Users/damianoneill/onos/tools/test/bin/ogroup-opts:12: command not found: complete
       /Users/damianoneill/onos/tools/test/bin/ogroup-opts:23: command not found: complete
       /Users/damianoneill/onos/tools/test/bin/ogroup-opts:34: command not found: complete
       /Users/damianoneill/onos/tools/test/bin/ogroup-opts:45: command not found: complete

At this point a set of aliases are defind, they can be viewed with 

      $ alias 

And a quick build can be kicked off with the following

      $ cd ~/onos; mcis 

At this point a release can be built by running an onos script 

      $ op 

This will write the release to the tmp directory for e.g. 

      ➜  onos git:(master) op
      -rw-r--r--  1 damianoneill  wheel    81M 18 Jun 09:43 /tmp/onos-1.3.0.damianoneill.tar.gz
      -rw-r--r--  1 damianoneill  wheel    81M 18 Jun 09:43 /tmp/onos-1.3.0.damianoneill.zip


# Running ONOS on dev machine
We'll use the onos-karaf and corresponding ok alias to run locally 

       $ export ONOS_IP=172.27.2.182; export ONOS_APPS=drivers,openflow,proxyarp,mobility,fwd; ok clean

# Loading a NETCONF (BTI 7800) Device into ONOS
Currently the learning of a new NETCONF Device by the platform is done via a configuration file.  

       cat ~/Applications/apache-karaf-3.0.3/etc/org.onosproject.provider.netconf.device.impl.NetconfDeviceProvider.cfg
       #
       # Instance-specific configurations, in this case, the number of
       # devices per node.
       #
       devConfigs = admin:admin@172.27.5.125:2022:active,cisco:cisco@192.168.56.20:2022:inactive,sdn:rocks@192.168.56.30:22:inactive
        
       #
       # Number of ports per device. This is global to all devices
       # on all instances.
       #
       # numPorts = 8
	 
I've added an entry for 172.27.7.125, once the configuration is completed we need to start ONOS and load the netconf feature bundle.


       onos> feature:install onos-netconf
       onos> list
       START LEVEL 100 , List Threshold: 50
       ID | State  | Lvl | Version          | Name
       ------------------------------------------------------------------------------
       40 | Active |  80 | 2.6              | Commons Lang
       41 | Active |  80 | 3.3.2            | Apache Commons Lang
       42 | Active |  80 | 1.10.0           | Apache Commons Configuration
       43 | Active |  80 | 18.0.0           | Guava: Google Core Libraries for Java
       ...
       174 | Active |  80 | 1.3.0.SNAPSHOT   | onos-netconf-provider-device

As you can see from the list command the last feature loaded was the netconf provider

You can confirm that the Device loaded correctly from the NETCONF configuration file with the following command

       onos> onos:devices
       id=netconf:admin@172.27.7.125:2022, available=true, role=MASTER, type=OTHER, mfr=, hw=, sw=, serial=

Or by tailing the log. 


