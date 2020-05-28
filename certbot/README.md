# How to automate SSL installation using Certbot
This tutorial uses certbot for automating SSL installation on apache web server

## Install Certbot on Fedora
Run the following command to install cerbot on fedora
```sh
$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```
```sh
$ sudo yum install certbot python2-certbot-apache
```

Check your certbot installation
```sh
$ sudo certbot --version
```

## Install SSL certificate for single domain
Run the following command to install SSL certifcate for a single domain, you can nominate your webroot folder using the `--webroot` parameter. Replace $EMAIL and $DOMAIN_NAME with your own values.
```sh
$ sudo certbot certonly --no-bootstrap  --email $EMAIL --agree-tos --webroot -w "/var/www/" -d "$DOMAIN_NAME"
```


## Install SSL certificate for wildcard domain
This section is only valid for certbot version 0.22+.

Run the following command for manual SSL certificate installation, add `--no-self-upgrade` if you don't want to certbot to upgrade during the installation 
```sh
$ sudo certbot certonly --manual --no-self-upgrade --no-bootstrap  --email $EMAIL --agree-tos  -d "domain.com" -d "*.domain.com" --manual-public-ip-logging-ok --preferred-challenges dns-01 --server https://acme-v02.api.letsencrypt.org/directory
```

## Renew SSL certificate
Run the following command for renew SSL certificate
```sh
sudo certbot --no-bootstrap renew
```
