[Updated 2020.05.25]

# Cloud From Scratch
Build a personal cloud system from scratch.

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

* Setup Domain
  * Register Domain (or use an existing one)
  * Cloudflare Setup

* Build Local Node
  * Configure OS
  * Wireguard
  * Docker
  * Caddy

* Setup Applications
  * Ghost
  * GOGS
  * Express

* Troubleshooting

For text editing we'll use vim, feel free to replace vim with the text editor of your choice.


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
We'll fill out clients-public-key in an upcomming step.

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

Configure cloudflare A record edge.example.com (replace .example.com with your domain name) point it to the IP of your VPS 

WITHOUT cloudflare proxy enabled.

SSL/TLS - Full (strict)
 
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

## Local Node
Get PI
PI (Amazon)
SD Card (Amazon)

Install Raspbian
note: if you don't have a pi, you could use a virtual machine, or laptop) if so note may need to adapt instructions.

download (raspbian buster lite at the time of this tutorial)

unzip
```
$ sudo dd if=2020-02-13-raspbian-buster-lite.img of=/dev/(yourdevicehere) bs=4M status=progress conv=fsync
or
- sudo unzip -p 2020-02-13-raspbian-buster.zip | dd if=2020-02-13-raspbian-buster-lite.img of=/dev/(yourdevhere) bs=4M status=progress conv=fsync
```
Do be careful with this command!  Be sure of your of=/dev/(yourdevhere) link.

enable ssh: place a file named ssh, without any extension, onto the boot partition before booting.
 
login

--user--: pi
--password--: raspberry

Initial Configuration
```
$ sudo raspbi-config
- set localization / keyboard
- set wifi connection (if not using ethernet)
- enable ssh [if not already enabled from boot ssh file]
```

Update system
```
$ sudo apt update
$ sudo apt upgrade
```

Wireguard

Check if already installed
```
$ which wg
```

If not try install
```
$ sudo apt install wireguard
```

If not try...
```
$ sudo apt-get install raspberrypi-kernel-headers

$ echo "deb http://deb.debian.org/debian/ unstable main" | sudo tee --append /etc/apt/sources.list.d/unstable.list

$ wget -O - https://ftp-master.debian.org/keys/archive-key-$(lsb_release -sr).asc | sudo apt-key add -

$ printf 'Package: *\nPin: release a=unstable\nPin-Priority: 150\n' | sudo tee --append /etc/apt/preferences.d/limit-unstable

$ sudo apt update

$ sudo apt install wireguard 

$ reboot
```
Verify
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

Create wg0.conf
```
$ vim wg0.conf
[Interface]
Address = 10.1.1.2/24
PrivateKey = xxxxx-your-private-key-here-xxxxx=

[Peer]
Endpoint = edge.example.com:51820
PublicKey = xxxxx-your-SERVERS-public-key-here-xxxxx=
AllowedIPs = 10.1.1.1/32, 10.1.1.2/32, 10.1.1.0/24
PersistentKeepalive = 25
```

IMPORTANT KEY PLACEMENTS
Replace 'edge.exmaple.com' with your edge.yourdomain.com setting from the cloudflare domain setting step.

Copy the PUBLIC key FROM your EDGE SERVER to the [Peer] PublicKey Here

Move wg0.conf to /etc/wireguard
```
$ sudo mv wg0.conf /etc/wireguard
```

Copy your PUBLIC key from your LOCAL HOST server to the publickey in your EDGE NODE wg0.conf file
```
(ON EDGE NODE)
$ vim /wireguard/etc/wg0.conf
[Peer]
#client 1 -- living room
PublicKey = this-local-node-public-key-goes-here=
AllowedIPs = 10.1.1.2/32
```

Continue on your local node... activate wg
```
$ sudo wg-quick up wg0
```

Verify
```
$ sudo wg
```
Should see up and down traffic (if no down traffic verify Endpoint address.)

Enable Wireguard to start automatically at boot
```
$ systemctl enable wg-quick@wg0.service
$ systemctl daemon-reload
$ systemctl start wg-quick@wg0
$ reboot ... (to check that is working)
$ systemctl status wg-quick@wg0
```

### Docker
```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

Verify
```
$ docker version
$ sudo docker info
$ sudo docker run hello-world
```

Create Network "Master"
```
$ docker network create -d bridge master
```


### Caddy 

$ vim ~/Caddyfile
http://example.com {
       respond "Yay!  It Works!"
       }

(replace example.com with your domain)

Setup Caddy Folders
```
$ mkdir ~/certs
$ mkdir ~/www
$ mkdir ~/www/example.com
```

Start Caddy
```
$ docker run -d --restart=always --name=caddy_web_server -p 80:80 -p 443:443 --network=master -v /home/pi/Caddyfile:/etc/caddy/Caddyfile -v /home/pi/www:/usr/share/caddy -v /home/pi/certs:/data caddy
```

Testing
```
$ sudo vim /etc/hosts
192.168.1.2 example.com
```
.. replace 192.168.1.2 with the ip of your local node.  and example.com with the domain you're using.

$ curl -v example.com
should see..
Yay!  It Works! 

### next: applications
