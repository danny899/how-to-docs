# How to install Jitsi Meeting on AWS EC2 (Debian OS)

## Introduction 
This document provides step-to-step guide on how to install Jitsi Meeting on Debian Operation System

## Prerequisites
- Debian OS 9 (Skretch) or Later
- A Domain name pointing to your server (e.g. meeting.yourdomain.com)

## Step 1. Set domain name
Go to your domain name host, and DNS provider, add an A DNS record pointing to your EC2 public IP.

## Step 2. Set hostname on server
SSH on to your EC2 server, edit the following files to include your domain name e.g. `meeting.yourdomain.com`

First edit the `/etc/hostname` file to ensure the domain `meeting.yourdomain.com` is included
```sh
$ sudo vi /etc/hostname
localhost meeting.yourdomain.com
```
Then, edit the `/etc/hosts` file 
```sh
$ sudo vi /etc/hosts
127.0.0.1	localhost meeting.yourdomain.com
```
Once this is set, you should be able to ping the hostname on the server.

## Step 3. Open ports in your firewall
Go to AWS Console, and go to the security group associated with your EC2 instance. Add the following rules to the SG:
- 80 TCP
- 443 TCP
- 10000 UDP

## Step 4. Install Jitsi Meet

### Add the Jitsi package library
Run following command as super user.
```sh
$ echo 'deb https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list
$ wget -qO -  https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
```
If you hit an error like `gnupg, gnupg2 and gnupg1 do not seem to be installed`, you will need to install `gnupg`
 or `gnupg2` using `apt-get install gnupg2`. Then you can add the key after that.
 
### Install Jitsi
```sh
# Ensure support is available for apt repositories served via HTTPS
$ apt-get install apt-transport-https

# Retrieve the latest package versions across all repositories
$ apt-get update

# Perform jitsi-meet installation
$ apt-get -y install jitsi-meet
```

During the installation, you will be asked to enter the hostname of the Jitsi Meet instance. If you have a Domain name in Step 1, enter it there. If you don't have a resolvable hostname, you can enter the IP address of the machine (if it is static or doesn't change).

This hostname (or IP address) will be used for virtualhost configuration inside the Jitsi Meet and also, you and your correspondents will be using it to access the web conferences.

### Install SSL Certificate
During the installation you will be asked to install a SSL Certificate. If you don't have a SSL cerificate for domain, and your DNS is set in step 1, select the **Generate a new self-signed certificate** option. Otherwise, you can install a SSL certificate manually later.

Once the installation is complete, Jitsi is up and running on the server

### Confirm that your installation is working
Launch a web browser (Chrome, Chromium or latest Opera) and enter the hostname `meeting.yourdomain.com` or IP address from the previous step into the address bar.

If you used a self-signed certificate (as opposed to using Let's Encrypt), your web browser will ask you to confirm that you trust the certificate.

You should see a web page prompting you to create a new meeting. Make sure that you can successfully create a meeting and that other participants are able to join the session.

If this all worked, then congratulations! You have an operational Jitsi conference service.

## Step 5. Secure conference creation using username and password
Once Jsiti is up and running on the server, you will have the following configuration files.

```sh
/etc/prosody/conf.avail/meeting.yourdomain.com.cfg.lua
/etc/jitsi/meet/meeting.yourdomain.com-config.js
/etc/jitsi/jicofo/sip-communicator.properties  //Only need this when you have SIP gateway installed
```

### Change Prosody configuration 
Edit the Prosody configuration file `/etc/prosody/conf.avail/meeting.yourdomain.com.cfg.lua`
a. Enable authentication for your main hostname
```sh
VirtualHost "meeting.yourdomain.com"
    authentication = "internal_plain"
```
b. Add new virtual host with anonymous login method for guests: (note you don't need to setup DNS for the guest.meeting.yourdomain.com domain)
```sh
VirtualHost "guest.meeting.yourdomain.com"
    authentication = "anonymous"
    modules_enabled = {
        "ping";
        "speakerstats";
        "turncreadentials";
        "conference_duration";
   }
    c2s_require_encryption = false
```

### Change Meet Config
Edit the following file`/etc/jitsi/meet/meeting.yourdomain.com-config.js`. Uncomment out the following two lines, and make sure the domain name is replace with your own domain, e.g. `meeting.yourdomain.com`
```sh
 // When using authentication, domain for guest users.
 anonymousdomain: 'guest.meeting.yourdomain.com',

 // Domain for authenticated users. Defaults to <domain>.
 authdomain: 'meeting.yourdomain.com',
```

### Change SIP-Communicator properties
Edit  `/etc/jitsi/jicofo/sip-communicator.properties` file
Add the `org.jitsi.jicofo.auth.URL=XMPP:meeting.yourdomain.com` to the file

## Step 6. Create an user for meeting
```sh
$ sudo prosodyctl register user [meeting.yourdomain.com] password
```

## Restart the service and Done
```sh
$ service jicofo restart
$ service jitsi-videobridge2 restart
$ service prosody restart
$ service nginx restart //only needed if running nginx
```
