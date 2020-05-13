# How to load balance Jitsi Meet with multiple Videobridge

## Introduction
In this document, we going to link the Jitsi meet installed in [previous article](./README.md) with multiple Videobridges.
Videobridge communicates using XMPP protocol, so we can setup a XMPP Pubsub node that listens to all Videobridge nodes, and pushes the events to Jicofo.

## Prerequisits
In this article, we assume you have two server machines, one (Server A) with default Jitsi Meet installation, and the other (Server B) is empty.

- Debian machine with [Default Jitsi Meet installation](README.md) (Server A)
- Debian machine OS 9 (Skretch) or later  (Server B)

## PART A. Configuration Change to Server A 
First, ssh onto Server A machine, again here we assume that you have Jitsi meet installed on Server A. You can refer to [this guide for default Jitsi Meet installation](README.md).

### Step 1. Add the JIDs of the Videobridges as Admins
Edit the Prosody configuration in `/etc/prosody/conf.d/<dnsname>.cfg.lua`, add the following line under `authentication = ...`

```sh
  admins = {
    "jitsi-videobridge.<dnsname>",
    "videobridge2.<dnsname>",
  }
```
So your configuration file should look like below.
```sh
VirtualHost "<dnsname>"
  -- enabled = false -- Remove this line to enable this host
  authentication = "internal_plain" //Note this value can be "anonymous", if you haven't enabled authentication
  admins = {
    "jitsi-videobridge.<dnsname>",
    "videobridge2.<dnsname>",
  }
  ...
```

### Configure Videobridge configuration
Now, edit the videobridge configuration in `/etc/jitsi/videobridge/config`, and look for the configuration line `JVB_OPTS="--apis=,"`, and change it to following. By default, the JID uses `jitsi-videobridge` as the default subdomain, so this change will set JID to use custom domain.
```sh
JVB_OPTS="--apis=rest,xmpp --subdomain=videobridge2" //????

JVB_HOST=<domainname> 
```

### Change the components interface from localhost to public interface

Edit the prosody configuration `/etc/prosody/prosody.cfg.lua` to listen for public IP, add the following configuration under `admin = { }`
```sh
component_ports = {5347}
component_interface = "<IPaddress>"  //replace this IP address with the public IP of your main Jitsi Meet server (Server A)
```

### Change Jicofo configuration to use public domain
Now, change the following configuration files to replace `localhost` with your jitsi domain.
First, `/etc/jitsi/jicofo/config`
```sh
JICOFO_HOST=<domainname>  //domain name is the domain name of your jitsi server (Server A)
```

### Change SIP-communicator configurations
In `/etc/jitsi/videobridge/sip-communicator.properties` to enable statistics and to set statistics to use pubsub, add the following lines if they are missing, or otherwise make sure the transport is pubsub
```sh
org.jitsi.videobridge.ENABLE_STATISTICS=true
org.jitsi.videobridge.STATISTICS_TRANSPORT=pubsub
org.jitsi.videobridge.PUBSUB_SERVICE=<domainname>
org.jitsi.videobridge.PUBSUB_NODE=sharedStatsNode
```

In `/etc/jitsi/jicofo/sip-communicator.properties` file, adding the following lines
```sh
org.jitsi.focus.pubsub.ADDRESS=<domainname>
org.jitsi.focus.STATS_PUBSUB_NODE=sharedStatsNode
```

Please note that so far we've been mainly using the domain name of the main Jitsi server (Server A).

### Add TCP Port 5347 on the main server (Server A)

## PART B. Changes on the second videobridge server (Server B)
SSH on to the second server

### Add the Jitsi package library
Run following command as super user.
```sh
$ echo 'deb https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list
$ wget -qO -  https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
```
If you hit an error like gnupg, gnupg2 and gnupg1 do not seem to be installed, you will need to install gnupg or gnupg2 using apt-get install gnupg2. Then you can add the key after that.

### Install jitsi videobridge
run the following command
```sh
$ apt-get update
$ apt-get install jitsi-videobridge
```

We will get prompted for Domain name of the main server, replace localhost with the domain name of the main server.

### Change the SIP-Communicator configurations
in `/etc/jitsi/videobridge/sip-communicator.properties` to enable statistics and to set statistics to use pubsub, add the following lines if they are missing, or otherwise make sure the transport is pubsub, make sure the `domainname` is pointing to server A.
```sh
org.jitsi.videobridge.ENABLE_STATISTICS=true
org.jitsi.videobridge.STATISTICS_TRANSPORT=pubsub
org.jitsi.videobridge.PUBSUB_SERVICE=<domainname>
org.jitsi.videobridge.PUBSUB_NODE=sharedStatsNode
```

???Not needed
In `/etc/jitsi/jicofo/sip-communicator.properties` file, adding the following lines
```sh
org.jitsi.focus.pubsub.ADDRESS=<domainname>
org.jitsi.focus.STATS_PUBSUB_NODE=sharedStatsNode
```

### Configure Videobridge configuration
Now, edit the videobridge configuration in `/etc/jitsi/videobridge/config`, and look for the configuration line `JVB_OPTS="--apis=,"`, and change it to following. By default, the JID uses `jitsi-videobridge` as the default subdomain, so this change will set JID to use custom domain.
```sh
JVB_OPTS="--apis=rest,xmpp --subdomain=videobridge2" 

JVB_HOST=<domainname> 
```

## PART C. Add components to main Jitsi server (Server A)

### Edit the Prosody configuration 
Edit the file in `/etc/prosody/conf.d/<dnsname>.cfg.lua`, at the bottom of the main `VirtualHost` add the following
```sh
component "videobridge2.<dnsname>" //This is the domain name of the second videobridge
component_secret = "<password>" //This can be found on second VB, under /etc/jitsi/videobridge/config
```
