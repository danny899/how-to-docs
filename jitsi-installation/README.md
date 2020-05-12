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
```sh
echo 'deb https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list
wget -qO -  https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
```

### Install Jitsi
```sh
# Ensure support is available for apt repositories served via HTTPS
apt-get install apt-transport-https

# Retrieve the latest package versions across all repositories
apt-get update

# Perform jitsi-meet installation
apt-get -y install jitsi-meet
```
