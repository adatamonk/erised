# ERISED - Only see what you want to see
### Adblocking using DNS-over-TLS


This is a project with the goal of blocking unwanted ads, trackers, etc across an entire operating system (web and apps). Currently focusing on Android since that's the only major OS with a native DNS-over-TLS implementation.

This group of containers will accept incoming DNS-over-TLS (DoH) requests, pass them through the [Pi-hole](https://pi-hole.net/) blocker, and then pass them upstream to an encrypted DNS server.

    Note:
    This currently deploys the Caddy container using a Caddy install using a Personal License. 
    Make sure to change this if you require the commerical license.

## Prerequsistes

Before deploying this you'll need to:

* get a KVM (or dedicated) server running Debian 9
* install [Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-9)
* install [Docker Compose](https://docs.docker.com/compose/install/)
* Get a DNS A or AAAA record pointing to your server's IP
* Open port 853 and 80 on your firewall (80 to get a certificate, and 853 for the actual service) 

## Server Prep

System & Dependencies:

`apt update && apt upgrade -y`

`apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common tmux apache2-utils ufw`

UFW:

`ufw allow OpenSSH`

`ufw allow http`

`ufw allow https`

`ufw allow 8080/tcp`

`ufw allow 853/tcp`

`ufw enable`

Docker:

`curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -`

`add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"`

`apt update`

`apt-cache policy docker-ce`

`apt install docker-ce`

Docker Compose:

`curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

`chmod +x /usr/local/bin/docker-compose`

## Deploying

Once you have all this, clone this repo

`cd /opt/`

`git clone https://github.com/eldridgea/erised.git`

Change into the cloned directory and run the build command replacing "YOUR_HOSTNAME" with your hostname and "YOUR_EMAIL" with your email.

`cd erised`

`docker-compose build --build-arg ERISEDHOST=YOUR_HOSTNAME --build-arg ERISEDEMAIL=YOUR_EMAIL`

**Note: sub-domains are not supported

Once this is done run 

`docker swarm init`

`docker-compose up -d`

(Or run `docker-compose up` if you want logging output to your terminal)


## Details

This will deploy a stack of three containers:

* cloudflared - to encrypt DNS queries before passing them upstream
* PiHole - to filter out unwanted results
* Caddy - to allow incoming DNS requests to use TLS

A DNS request will look like this:

        ---------
        |Your   |
        |Device |
        |       |
        ---------
            |
            |
            |
            |
        ---------
        |       |
        |Caddy  |
        |       |
        ---------
            |
            |
            |
            |
        ---------
        |       |
        |pihole |
        |       |
        ---------   
            |
            |
            |
            |
        ---------------
        |             |
        | cloudflared |
        |             |
        ---------------
            |
            |
            |
            |
        -------------------
        |                 |
        | Cloudflare's    |
        | 1.1.1.1 service |
        -------------------

## Using on Android 9.0 +
DNS-over-TLS is a built in feature on Android 9 (Pie) and later. If used, Android will use that DNS server for all requests on all networks. So this allows for system-wide blocking on all apps and all networks without the need for an always-on VPN.

To use this, open up your Android settings > Network & Internet > Advanced > Private DNS.

Select the "Private DNS provider hostname" option and enter in your hostname that you used for your DNS record and in the build command.
