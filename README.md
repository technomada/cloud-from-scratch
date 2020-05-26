[Updated 2020.05.25]

# Cloud From Scratch
Build your own personal cloud system from scratch.

You may be surpized to find that building and maintaining your own cloud is not only not difficult, but really fun and rewarding. The following instructions should provide everything you need to build a multi host capable personal cloud computing platform from scratch. Using primarily existing technologies and inspired by the "Arch Way" (Simplicity, Modernity, Pragmatism, User centrality, Versatility) this cloud design project aims to require a small amount of technical attention and allow a wide range of functionality. After completing the following steps you'll have yourself a fully functional personal cloud system ready to be shaped into the services you wish. I don't know about you, but I know I'm excited, so let's get started.


# Overview
When completed the system looks something like this.

```
[ EDGE NODE VPS ]                     |               [ LAN NODE ]
                                      |               
    [Wireguard] <---------------------+-------------- [Wireguard]
         ^                            |                   ^
         |                            |                   |
         v                            |                   v
    [Caddy Reverse Proxy]             |         +----------------------------------+
                                      |         | DOCKER                           |
                                      |         |                                  |
                                      |         |    [ ------ Caddy ---------]     |
                                      |         |       |       |        |         |
                                      |         |       v       |        v         |
                                      |         |     [Ghost]   |    [Express]     |
                                      |         |               v                  |
                                      |         |             [GOGS]               |
                                      |         |                                  |
                                                +----------------------------------+
                                                
                                                
         ** Internet  **                                  ** Home Network **

```

## Sections
* Provision Edge Node
  * VPS Instance
  * Wireguard
  * Docker
  * Caddy
  
* Build Local Node
  * Configure OS
  * Wireguard
  * Docker
  * Caddy

* Setup Domain
  * Register Domain (or use an existing one)
  * Cloudflare Setup

* Setup Applications
  * Ghost
  * GOGS
  * Express

* Troubleshooting

## Provision Edge Node
Create a VPS instance at your favorate VPS service, like Digital Ocean or Vultr.  [use these affiliate links to support this project: [Digital Ocean](https://digitalocean.com) | [Vultr](https://vultr.com).

Any level >= 512MB will likely do. Create a new instance using Debian 10 (Buster)

Log in via SSH to your new server.

```
$ apt update
$ apt upgrade
```

### Wireguard

("Users with Debian releases older than Bullseye should enable backports." This includes Buster so, we'll do backport.)


Edit sources.list, add buster-backports, install wireguard.

```
$ vim /etc/apt/sources.list.d/sources.list
  deb https://deb.debian.org/debian buster-backports main

$ apt update
$ apt -t buster-backports install "wireguard"

$ reboot
```

Verify Install
```
$ which wg
```

Generate Wireguard Keys
```
$ cd ~
$ mkdir wg-setup
$ cd wg-setup

$ wg genkey | tee privatekey | wg pubkey > publickey
```

Create wg0.conf file
```
$ vim wg0.conf
[Interface]
Address = 10.1.1.1/24
ListenPort = 51820
PrivateKey = your-private-key=

[Peer]
#client 1 -- living room
PublicKey = clients-public-key=
AllowedIPs = 10.1.1.2/32

[Peer]
#remote pi 1 -- kitchen
PublicKey = clients-public-key= 
AllowedIPs = 10.1.1.3/32

$ cp wg0.conf /etc/wireguard/wg0.conf
```

Start Wireguard
```
$ wg-quick up wg0
```

Verify Looks good
```
$ wg
```

Enable Wireguard to start automatically at boot
```
$ systemctl enable wg-quick@wg0.service
$ systemctl daemon-reload
$ systemctl start wg-quick@wg0
$ reboot ... (to check that is working)
$ systemctl status wg-quick@wg0
```

Test
```
$ ip a ... (should show wg interface)
$ wg   ... (should show client config)
```

### Docker
https://docs.docker.com/engine/install/debian/

```
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
$ apt update
$ apt install docker-ce docker-ce-cli containerd.io
```

Test (should download and run docker hello world image)
```
$ docker run hello-world  
```


### Caddy
https://hub.docker.com/_/caddy


Create Caddyfile
```
$ vim ~/Caddyfile
example.com:80 {
        #tls letsencrypt@example.com

        #reverse_proxy 10.1.1.2:80
	      respond "It Works!!"
                }
```
(we'll replace example.com with your domain later in the domain step)

Create a network called 'master'
```
$ docker network create -d bridge master
```

Setup Caddy file locations
```
$ mkdir ~/certs
$ mkdir ~/www
$ mkdir ~/www/example.com
```

Start Caddy
```
$ docker run -d --restart=always --name=caddy_web_server -p 80:80 -p 443:443 --network=master -v /root/Caddyfile:/etc/caddy/Caddyfile -v /root/www:/usr/share/caddy -v /root/certs:/data caddy
```

## Setup Domain
### Domain Name
Use your own existing domain or register a new one.  | [namecheap](https://namecheap.com) -- support this project by using this affiliate link.
	
### Cloudflare
	Use [CloudFlare](https://cloudflare.com) as your name server (set your domain name name servers as your cloudflare account instructs.)
	Configure cloudflare A record to point to the IP of your VPS.  With cloudflare proxy enabled.
	SSL/TLS
		- Full (strict)
 
Modify Edge Node Caddyfile
```
$ vim ~/Caddyfile
example.com:80 {
        tls certs@example.com

        #reverse_proxy 10.1.1.2:80
	      respond "It Works!!"
                }
```
Replace 'example.com' with your domain name.  Replace 'certs@example.com' with your domain's admin email address.

In future references to 'example.com' replace 'example.com' with your domain name.

Restart Caddy
```
$ docker restart caddy_web_server
```

Check your domain
```
$ curl -v http://example.com
```
It should show "It Works!!"
