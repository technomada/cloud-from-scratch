[ Tested 2020.05.28 ] [ Updated 2020.05.28 ] 

# Cloud From Scratch
Build a self-hosted personal private cloud system from scratch.

You may be surprized to find that building and maintaining your own private cloud system, using your own equipment, hosted in your own home, is not only possible, it's relatively easy and is really fun and rewarding. The following instructions should provide everything you need to build a multi-host capable personal cloud computing platform from scratch. Using primarily off the shelf technologies and inspired by the "Arch Way" (**Simplicity, Modernity, Pragmatism, User centrality, Versatility**) this project aims to require only a small amount of technical attention and allow a wide range of expression and functionality. After completing the following steps you'll have yourself a fully functional personal private cloud system ready to be shaped into the services you wish.  I don't know about you, but I know I'm excited, so let's get started.


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
                                      |         |				   |
				      |		+----------------------------------+
				      |		.				   .
                                      |         .    [ ------ Caddy ---------]     .
                                      |         .        |       |        |        . 
                                      |         .        v       |        v        .
                                      |         .       [APP]    |      [APP]      . 
                                      |         .                v                 . 
                                      |         .              [APP]               . 
                                      |         .                                  . 
                                                +. . . . . . . . . . . . . . . . . +
                                               
 
         ** Internet  **                               ** Home Network Cloud **

```

From this platform you'll be able to install, own, operate and access from anywhere, from any device, any cloudable software you choose to install.

Chat * Photos * Calls * File Storage * Music * Notes * Weather * News * Etc

You'll have your own public domain address let's call it https://example.com resolving data and apps from inside a cloud system you built running on your own home network. 

Ok, Let's get started.

## Requirements
For this setup you'll need...
* **Cloudflare Account**
* **Domain Name**
* **Raspbery PI** (v3 or greater, or adopt the instructions to a PC or Virtual Machine)
* **VPS Account** (Linode/Digital Ocean/Vultr)

If you're not already familiar with [Wireguard](https://www.wireguard.com/) and [Docker](https://www.docker.com/) you may want to first familiarize yourself with these as they play core roles in this project. These instructions assume you are comfortable with command line based installation and configuration.  For text editing we'll use vim, feel free to replace vim with the text editor of your choice.

Prefer something a little more automated?  Consider one of [these](#web-gui-automated-run-your-own-cloud-system).

## Sections
* **Provision Edge Node**
  * VPS Instance
  * Wireguard
  * Docker
  * Caddy

* **Setup Domain**
  * Register Domain (or use an existing one)
  * Cloudflare Setup

* **Build a Local Node**
  * Configure OS
  * Wireguard
  * Docker
  * Caddy

* **Example Applications**
  * Ghost
  * GOGS
  * Express

* Troubleshooting
* Optional Configurations
* Discussion
* Links


## Provision Edge Node
The edge node functions as a lightweight, always online, public access gateway, mainly routes traffic, provides a layer of privacy and helps mitigate NAT issues.

Create a VPS instance at your favorate VPS service, like Digital Ocean or Vultr.  (use these affiliate links to support this project: [Digital Ocean](https://digitalocean.com) | [Vultr $100 free credit for 30 days](https://www.vultr.com/?ref=8580218-6G).)

Any tier level with at least **512MB RAM** should be enough.

Create a new instance using **Debian 10** (Buster)

For the purpose of this tutoral we'll assume your edge node ip address is `198.51.100.1`.

Log in via SSH to your new server
```
$ ssh root@your-new-server-ip
```

and update the system
```
$ apt update
$ apt upgrade
```

### Wireguard

("Users with Debian releases older than Bullseye should enable backports." This includes Buster so, we'll do backport.)


Edit sources.list, add buster-backports, install wireguard.

```
$ vim /etc/apt/sources.list.d/sources.list
```

add
```
deb https://deb.debian.org/debian buster-backports main
```

install wireguard
```
$ apt update
$ apt -t buster-backports install "wireguard"

$ reboot
```
Reconnect to server after reboot.

Verify Install
```
$ which wg
```
(should see something like `/usr/bin/wg`)

Generate Wireguard Keys
```
$ cd ~
$ mkdir wg-setup
$ cd wg-setup

$ wg genkey | tee privatekey | wg pubkey > publickey
```
(contents of privatekey or public key file look like `ayQFJiofd+fji2oDi8N/Jfi3=`)

Create wg0.conf file
```
$ vim wg0.conf
```
We're using `10.1.1.1/24` from here forward you may use this or your own preference for the wireguard network.  ListenPort may also be your choice.  Here we'll use `51820`.
```
[Interface]
Address = 10.1.1.1/24
ListenPort = 51820
PrivateKey = your-private-key-text-from-privatekey-genearation=

#[Peer]
##client 1 -- living room
#PublicKey = clients-public-key-from-a-future-step=
#AllowedIPs = 10.1.1.2/32

#[Peer]
##remote pi 1 -- kitchen
#PublicKey = clients-public-key-from-a-future-step= 
#AllowedIPs = 10.1.1.3/32
```

Copy the `~/wg-setup/wg0.conf` file to `/etc/wireguard/wg0.conf`
```
$ cp wg0.conf /etc/wireguard/wg0.conf
```

We'll come back and fill out clients-public-key in the `/etc/wireguard/wg0.conf` file in an upcomming step.

Start Wireguard
```
$ wg-quick up wg0
```

Should see something like
```
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.1.1.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

Verify Looks good
```
$ wg
```
should see someting like
```
interface: wg0
  public key: sf8fjDJj9fiJaawQdfiDFi8sfjkl+fxXIxf/=
  private key: (hidden)
  listening port: 51820
```

Enable Wireguard to start automatically at boot
```
$ systemctl enable wg-quick@wg0.service
```

Reboot to verify its working.
```
$ reboot
```

Verify
```
$ systemctl status wg-quick@wg0
```

Test
```
$ ip a ... (should show wg0 interface)
$ wg   ... (should show client config)
```


### Docker
https://docs.docker.com/engine/install/debian/

Install Docker
```
$ apt install gnupg2 software-properties-common
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
$ apt update
$ apt install docker-ce docker-ce-cli containerd.io
```

Test (this should download and run docker hello world image)
```
$ docker run hello-world  
```


### Caddy
https://hub.docker.com/_/caddy


Create Caddyfile
```
$ vim ~/Caddyfile
example.com:80 {
        #tls cert@example.com

	#reverse_proxy 10.1.1.2:80
	
	respond "It Works!!"
        }
```
(we'll replace `example.com` with your domain later in the domain step)

Create a docker network called `master`
```
$ docker network create -d bridge master
```

Create caddy `certs` location.
```
$ mkdir ~/certs
```

Start Caddy
```
$ docker run -d --restart=always --name=caddy_web_server -p 80:80 -p 443:443 --network=master -v /root/Caddyfile:/etc/caddy/Caddyfile  -v /root/certs:/data caddy
```

Check
```
$ docker ps -a
```
should see something like
```
eb631324c3b9        caddy               "caddy run --config …"   22 seconds ago      Up 21 seconds              0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 2019/tcp   caddy_web_server
```

## Setup Domain
### Domain Name
Use your own existing domain or register a new one.  | [namecheap](https://namecheap.com) -- support this project by using this affiliate link.
	
### Cloudflare
Use [CloudFlare](https://cloudflare.com) as your name server (set your domain name name servers to the nameserver names your cloudflare account instructs.)

**DNS**
Configure an A record to point to the IP of your VPS.  **WITH cloudflare proxy** enabled. eg `A @ 198.51.100.1`
(replace `example.com` with your domain name and `198.51.100.1` is an example address, use the ip address of your edge node whenever you see the `198.51.100.1` address.)

Configure cloudflare a CNAME record `edge.example.com` (replace `.example.com` with your domain name eg `edge.yourdomain.com`) point it to your domain name **WITHOUT cloudflare proxy (click to make a grey cloud)**. eg `CNAME edge example.com`

**SSL/TLS**
Set to **Full (strict)** Option  (otherwise you may get too many an redirects error.)
  
Modify Edge Node Caddyfile
```
$ vim ~/Caddyfile
example.com:80 {
        #tls certs@example.com

        #reverse_proxy 10.1.1.2:80
	respond "It Works!!"
                }
```
Replace `example.com`s with your domain name.  Replace 'certs@example.com' with your domain's admin email address.

In future references to `example.com` replace `example.com` with your domain name.

Restart Caddy
```
$ docker restart caddy_web_server
```

Check your domain
```
$ curl -v http://example.com
```
should see something like
```
> GET / HTTP/1.1
> Host: example.com
> User-Agent: curl/7.64.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Thu, 28 May 2020 21:56:06 GMT
< Content-Length: 10
< Connection: keep-alive
< Set-Cookie: __cfduid=d7b74165a55e4ebe027d2a259181e53731590702966; expires=Sat, 27-Jun-20 21:56:06 GMT; path=/; domain=.example.com; HttpOnly; SameSite=Lax
< CF-Cache-Status: DYNAMIC
< cf-request-id: 02fee21e7b0000920a16007200000001
< Server: cloudflare
< CF-RAY: 59ab3943faf7920a-EWR
< 
* Connection #0 to host example.com left intact
It Works!!
```
if not, check your Caddyfile then restart caddy.
if still not... perhaps the caddy log will help...
```
$ docker logs caddy_web_server
```

Enable https
```
$ vim Caddyfile
```

Remove `:80` and uncomment your email address.
```
example.com {
        tls certs@example.com

        #reverse_proxy 10.1.1.2:80
	respond "It Works!!"
                }
```

Restart Caddy
```
$ docker restart caddy_web_server
```

Make sure Certificate obtained successfully (use ctrl+c to return to term)
```
$ docker logs -f caddy_web_server
```

Check your domain
```
$ curl -v https://example.com
```
Should see It Works!! and TLS handshakes.

GOOD JOB!  At this point you have a working edge node with a publicly assigned domain name ready to accept and forward web traffic to your at home local network.  Now might be a good moment to take a break and go for a little walk.  Next up... local node!


## Local Node
Local nodes live within your home network.  In this system local nodes are pretty much where everything lives and happens.  Cloud systems can be built from one or more hosts, but to keep things simple we'll start out with just one node, a Raspberry Pi.

Get Pi

Pi [Kit $100](https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9) This is a kit, feel free to get any Pi setup v3 or better.

Install Raspbian
Note: if you don't have a Pi, you could use a virtual machine, or laptop if so note you may need to adapt these instructions to your situation.

Download (Raspbian Buster Lite at the time of this tutorial)

Create the bootable sdcard

**Be careful with this step!  Don't delete things you don't want to delete.  Be sure of your of=/dev/(yourdevhere) link.**

These commands may help you locate your target.
```
$ df -h
$ lsblk
```

Copy the raspbian zip file onto sdcard (this will erase everything on the sdcard)  dd is one option, use a disk imaging tool you're comfortable with.
```
$ unzip 2020-02-13-raspbian-buster-lite.zip
$ sudo dd if=2020-02-13-raspbian-buster-lite.img of=/dev/(yourdevicehere) bs=4M status=progress conv=fsync
```
**or** the same thing in one step
```
$ unzip -p 2020-02-13-raspbian-buster.zip | sudo dd of=/dev/(yourdevhere) bs=4M status=progress conv=fsync
```
Note `2020-02-13-raspbian-buster.zip` file name may be different for you.

Enable ssh: place a file named ssh, without any extension, onto the boot partition before booting.

(optionally, enable ssh)
Open a terminal to the boot partion then...
```
$ touch ssh
```

(optionally to find the ip address of your pi (Note: you'll need to use a keyboard and monitor if doing a wireless setup.)
```
$ sudo arp-scan 192.168.1.0/24 -I eth0
```
Assuming you have `arp-scan` installed and your nework is `192.168.1.0/24` and your pi is connected on the same network as your `eth0` connetion.


 
login

**user** pi

**password** raspberry

Initial Configuration
```
$ sudo raspi-config
```
- set password
- set localization / keyboard
- set wifi connection (if not using ethernet)
- enable ssh [if not already enabled from boot ssh file]

If you're using ssh it's a good idea to set only key based ssh.

note: once you're connected with wired or wireless you may find ssh easier to continue from here for copy/paste of commands.

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

$ sudo reboot
```
Verify
```
$ which wg
```
Should see something like
`/usr/bin/wg`

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
```

Configure, remember to replace the keys and domain name `edge.example.com` with your own.
```
[Interface]
Address = 10.1.1.2/24
PrivateKey = your-LOCAL-NODE-private-key-goes-here=

[Peer]
Endpoint = edge.example.com:51820
PublicKey = your-EDGE-NODE-public-key-goes-here=
AllowedIPs = 10.1.1.1/32, 10.1.1.2/32, 10.1.1.0/24
PersistentKeepalive = 25
```

IMPORTANT KEY PLACEMENTS
Replace `edge.exmaple.com` with your `edge.yourdomain.com` setting from the cloudflare domain setting step.

You'll want to follow this closely it can get tricky...


Copy the **PRIVATE** key of your **LOCAL NODE** to the [Interface] PrivateKey `your-LOCAL-NODE-private-key-goes-here`

Copy the **PUBLIC** key of your **EDGE NODE** to the [Peer] PublicKey `your-EDGE-NODE-public-key-goes-here`

Move wg0.conf to /etc/wireguard
```
$ sudo mv wg0.conf /etc/wireguard
```


(on **EDGE NODE**)
```
$ vim /wireguard/etc/wg0.conf
```

Copy your **PUBLIC** key of your **LOCAL NODE** to the `your-LOCAL-NODE-public-key-goes-here` in your **EDGE NODE** wg0.conf file.
Uncomment '#' 
```
[Peer]
#client 1 -- living room
PublicKey = your-LOCAL-NODE-public-key-goes-here=
AllowedIPs = 10.1.1.2/32
```

(on **EDGE NODE**) restart wireguard
```
$ wg-quick down wg0
$ wg-quick up wg0
```

(on **EDGE NODE**) Check
```
$ wg
```
Should see the something like this (notice the 10.1.1.2/32 entry

```
interface: wg0
  public key: axxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
  private key: (hidden)
  listening port: 51820

peer: Oyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyys=
  allowed ips: 10.1.1.2/32
```

Continue on your **LOCAL NODE**... activate wg
```
$ sudo wg-quick up wg0
```
should see
```
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.1.1.2/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
```

Verify
```
$ sudo wg
```
Should see up and down traffic (if no down traffic verify Endpoint address. if still no, check your keys on both edge and local node wg0.conf files.)

When connected..
on the edge node `$ wg` should look something like
```
interface: wg0
  public key: aaaaaaaaaaaaaaaaaaaaaaaaaaaaa=
  private key: (hidden)
  listening port: 51820

peer: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb=
  endpoint: 299.12.133.4:52001
  allowed ips: 10.1.1.2/32
  latest handshake: 10 seconds ago
  transfer: 180 B received, 92 B sent
```

on the local node `$ sudo wg` should look something like
```
interface: wg0
  public key: ccccccccccccccccccccccccccccc=
  private key: (hidden)
  listening port: 52001

peer: dddddddddddddddddddddddddddddddddd=
  endpoint: 445.176.4.3:51820
  allowed ips: 10.1.1.1/32, 10.1.1.2/32, 10.1.1.0/24
  latest handshake: 3 seconds ago
  transfer: 92 B received, 1.33 KiB sent
  persistent keepalive: every 25 seconds

```
Be sure to note that the received has > 0 bytes.  It may look connected when it's not if received is 0 B.

continuing on **LOCAL NODE**
Enable Wireguard to start automatically at boot
```
$ sudo systemctl enable wg-quick@wg0.service
```

Test
```
$ sudo reboot
$ systemctl status wg-quick@wg0
$ ip a
$ ping 10.1.1.1
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
$ sudo docker network create -d bridge master
```


### Caddy 
```
$ vim ~/Caddyfile
http://example.com {
       respond "Yay!  It Really Works!"
       }
```
(replace `example.com` with your domain)

Create caddy folders
```
$ mkdir ~/certs
$ mkdir  -p ~/www/example.com
```
(replace `example.com` with your domain name)

Start caddy
```
$ sudo docker run -d --restart=always --name=caddy_web_server -p 80:80 -p 443:443 --network=master -v /home/pi/Caddyfile:/etc/caddy/Caddyfile -v /home/pi/www:/usr/share/caddy -v /home/pi/certs:/data caddy
```

Testing
```
$ sudo vim /etc/hosts
127.0.0.1 example.com
```
.. replace  `example.com` with the domain you're using.

```
$ curl -v example.com
```
You should see.. Yay!  It Really Works! 

if no try learning more with...
```
$ sudo docker ps -a
$ sudo docker logs caddy_web_server
```

Once local is working try globally

on **EDGE NODE**
```
$ vim ~/Caddyfile
```
uncomment reverse proxy, comment respond
```
example.com {
	tls cert@example.com
	
	reverse_proxy 10.1.1.2:80
	
	#respond "It Works!!"
	}
```

Restart caddy
```
$ docker restart caddy_web_server
```

back on **LOCAL NODE**

comment out test local host mapping
```
$ sudo vim /etc/hosts
```
```
#127.0.0.1 example.com
```


Test (note the 's' in https://)
```
$ curl -v https://example.com
```
you should see.
`Yay!  It Really Works!!`

SUPER!! Everything is connected!  You now have a public domain which is being served from a device within your local network.  Cool.  Next let get's get that node doing things.

## Applications
A cloud needs to do things, you'll be able to build your cloud into whatever suits you. Here you'll find a few examples to get you started, we'll start with a simple static file server then move on to dynamic applications.

### Caddy static file serving

(note from here out you can assume we're dealing with the **LOCAL NODE**)

Edit Caddyfile
```
$ vim ~/Caddyfile
```

Comment out the `respond` line and add `file_server` section.
```
http://example.com {
	
	file_server {
		root /usr/share/caddy/example.com
		}
	#respond "Yay!  It really works!"	
	}
```

Edit ~/www/example.com/index.html
```
$ vim ~/www/example.com/index.html
```

Say something witty.
```
Static works too!!
```


Restart Caddy
```
$ sudo docker restart caddy_web_server
```


Test
```
$ curl -v https://example.com
```
Try testing from your browser on your phone or desktop.

Browse to [https://example.com]

You should see your wit!!

Here you might notice too that if you enter http://example.com you'll be forward to the secure link https://example.com.


Fine work.  Now for something more appy.


### Dynamic (Sky's the limit) applications.

Here's few apps setups to try, click one of these or jump to the [more](apps/README.md) page for expanded lists.

* [Ghost](apps/ghost.md)
* [GOGS](apps/gogs.md)
* [Express](apps/express.md)

There's plenty [more](apps/README.md) one can install, and more becomes available all the time.

There you have it.  Your own cloud.  Let us know what you do with yours!

## Troubleshooting

## Discussion

Why run a personal cloud.
* Ad Free (Calmer/Cleaner/Faster)
* Control
* Convienence
* Persistance
* Personaliztion
* Privacy
* Options
* Speed
* "Unlimited" Storage

### Related Links

* https://www.inkandswitch.com/local-first.html
* https://www.reddit.com/r/selfhosted/
* https://neustadt.fr/essays/against-a-user-hostile-web/

### Similar Projects
* https://github.com/ahmadsayed/cloud-from-scratch
* https://github.com/funkypenguin/geek-cookbook
* https://github.com/progmaticltd/homebox
* https://github.com/sovereign/sovereign
* https://freedombox.org/


### Web GUI automated run your own cloud systems
Prefer turnkey web gui based management?  Take a look at these.
* https://caprover.com/
* https://cloudron.io/
* https://cozy.io/
* https://nextcloud.com/
* https://owncloud.org/
* https://sandstorm.io/
* https://yunohost.org/

### Useful services
* https://www.backblaze.com/
* https://ngrok.com/
* https://www.zerotier.com/


Have a suggestion, question, comment, or request?  Submit a [new issue](https://github.com/technomada/cloud-from-scratch/issues/new).
