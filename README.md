[ Tested 2020.05.28 ] [ Updated 2020.06.03 ] [ Design Version 0.1 (Beta) ]

Forward

This project is very early in, it's mission is to maintain contemporary patterns and resources including specific instruction to create and operate personal cloud systems.  If you find a 'bug' or have ideas please contribute with a new issue or by making a pull request.  Thanks!  Let's make this a great way to build our own cloud.


# Cloud From Scratch
Build a self-hosted personal private cloud system from scratch.

You may be surprised to find that building and maintaining your own private cloud system, using your own equipment, hosted in your own home, is not only possible, it's relatively easy and is really fun and rewarding. The following instructions should provide everything you need to build a multi-host capable personal cloud computing platform from scratch. Using primarily off the shelf technologies and inspired by the "Arch Way" (**Simplicity, Modernity, Pragmatism, User centrality, Versatility**) this project aims to require only a small amount of technical attention and allow a wide range of expression and functionality. After completing the following steps you'll have yourself a fully functional personal private cloud system ready to be shaped into the services you wish.  I don't know about you, but I know I'm excited, so let's get started.


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

You'll have your own public domain name like https://bobspersonalwebsite.com resolving data and apps from a cloud system you built from scratch running on equipment inside your own home network. 

Cool, let's do it!

## Requirements
For this particular setup we'll use a...
* **Cloudflare Account** - Bandwidth Assistance
* **Domain Name** - Public Address
* **Raspberry PI** - Local Host (v3 or greater, or adopt the instructions to a PC or Virtual Machine)
* **VPS Account** - Privacy and NAT Mitigation (Linode/Digital Ocean/Vultr/Etc)

Note the requirements are needed for following the directions specifically, if you're comfortable doing something different feel free to adapt the instruction to suit your situation.  For example, depending on your drivers you may or may not feel comfortable using Cloudflare.  Some advantages of Cloudflare are privacy (hiding your actual server IP (if you proxy all the addresses,)) caching (if you expect a lot of traffic,) nice API control, and it's free.  But, you trade that for unencrypted man in the middle, certainly an understandable deal-breaker for certain situations.  If you find yourself in this situation Cloudflare can be skipped and your domain mapped directly to your edge node IP using your registrars dns panel.    

If you're not already familiar with [Wireguard](https://www.wireguard.com/) and [Docker](https://www.docker.com/) you may want to first familiarize yourself with these as they play core roles in this project. These instructions assume you are comfortable with command-line based installation and configuration.  For text editing we'll use vim, feel free to replace vim with the text editor of your choice.

If this sounds like your cup of tea dear reader read on. Or if you prefer something a little more automated consider one of [these](#web-gui-automated-run-your-own-cloud-system).

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
The edge node functions as a lightweight, always online, public access gateway, mainly routes traffic, provides a layer of privacy and mitigates NAT issues.

Create a VPS instance at your favorite VPS service, like Digital Ocean or Vultr.  (use these affiliate links to support this project: [Digital Ocean](https://digitalocean.com) | [Vultr $100 free credit for 30 days](https://www.vultr.com/?ref=8580218-6G).)

Any tier level with at least **512MB RAM** should be enough.

Create a new instance using **Debian 10** (Buster)

For the purpose of this tutorial we'll assume your edge node ip address is `198.51.100.1`.

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
We'll use wireguard to route traffic through your NAT and provide a layer of privacy to keep your home IP address private.  You can think of this kinda like a reverse VPN.

[Install and configure wireguard](edge-node/edge-node-debian-10-wireguard.md) on edge node.

### Docker
Running services in docker keeps things tidy and manageable, we'll setup docker to contain our edge services starting with a reverse proxy web server.

[Install and configure docker](edge-node/edge-node-debian-10-docker.md) on edge node.

### Caddy
Caddy is super easy to use, automatically supports Let's Encrypt https certs and will be used to route our domain requests into our home node network.

[Install and configure caddy](edge-node/edge-node-debian-10-caddy.md) on edge node.


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
Set to **Full (strict)** Option  (otherwise you may get a 'too many redirects' error.)
  
[Configure your domain name](edge-node/edge-node-debian-10-configure-domain-name.md) on the edge node.

GOOD JOB!  At this point you have a working edge node with a publicly assigned domain name ready to accept and forward web traffic to your at home local network.  Now might be a good moment to take a break and go for a little walk.  Next up... local node!


## Local Node
Local nodes live within your home network.  In this system local nodes are pretty much where everything lives and happens.  Cloud systems can be built from one or more hosts, but to keep things simple we'll start out with just one node, a Raspberry Pi.

### Get Pi

Pi [Kit $100](https://www.amazon.com/CanaKit-Raspberry-4GB-Starter-Kit/dp/B07V5JTMV9) This is a kit, feel free to get any Pi setup v3 or better.

### Install Raspbian
Note: if you don't have a Pi, you could use a virtual machine, or laptop if so note you may need to adapt these instructions to your situation.

[Install and configure raspbian](local-node/local-node-raspbian-setup.md) on your local node.

### Wireguard
Here we'll setup the local side wireguard.  When the local node boots and gets an internet connection it will automatically connect to your edge node and become virtually accessible to external requests.

[Install and configure wireguard](local-node/local-node-raspbian-wireguard.md) on your local node.

### Docker
Local node services are setup within docker containers.  Web requests are routed through caddy (running in a container) to other containerized processes but are all mapped into urls within your domain name. 

[Install and configure docker](local-node/local-node-raspbian-docker.md) on your local node.

### Caddy 
When a web request arrives at your edge node, it hits your edge node caddy instance, which routes the request through wireguard to your local caddy which then routes it to the docker container running the service at that url.

[Install and configure caddy](local-node/local-node-raspbian-caddy.md) on your local node.


SUPER!! Everything is connected!  You now have a publicly addressable domain name which serving content from a device within your local network.  Cool.  Next let get that node doing things.

## Applications
A cloud needs to do things, you'll be able to build your cloud into whatever suits you. Here you'll find a few examples to get you started, we'll start with a simple caddy static file server then move on to dynamic container based applications.

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

Here you might notice too that if you enter http://example.com you'll be forwarded to the secure link https://example.com.


Fine work.  Now for something more appy.


### Dynamic (Sky's the limit) applications.

Here's few apps setups to try, click one of these or jump to the [more](apps/README.md) page for expanded lists.

* [Ghost](apps/ghost.md)
* [GOGS](apps/gogs.md)
* [Express](apps/express.md)

There's plenty [more](apps/README.md) one can install, and more becomes available all the time.

There you have it.  Your own cloud.  Let us know what you do with yours!

## Troubleshooting
Sometimes things change or there might be a misstyped command.  Here are some hints and tips to get the wrong back in line.

## Doing More

## Security Adjustments

## Discussion

Why run a personal cloud.
* Ad Free (Calmer/Cleaner/Faster)
* Control
* Convenience
* Persistence
* Personalization
* Privacy
* Options
* Speed
* "Unlimited" Storage

## Where to go next
* add TC (traffic control)
* multiple nodes
* email hosting
* https access lan side
* advanced management
	* automated application distribution
	* system health monitoring
	* network boot and automated dynamic provisioning
* remote service sharing and integration
* firewall
* cloud suite


### Related Links
* https://github.com/cncf/landscape#trail-map
* https://www.reddit.com/r/selfhosted/
* https://redecentralize.org/
* https://www.cncf.io/projects/

### Similar Projects
* https://github.com/ahmadsayed/cloud-from-scratch
* https://github.com/awesome-selfhosted/awesome-selfhosted#self-hosting-solutions
* https://github.com/funkypenguin/geek-cookbook
* https://github.com/progmaticltd/homebox
* https://github.com/sovereign/sovereign
* https://freedombox.org/

### Additional Reading
* https://medium.com/better-programming/running-a-container-with-a-non-root-user-e35830d1f42a
* https://neustadt.fr/essays/against-a-user-hostile-web/
* https://www.howtoforge.com/tutorial/build-your-own-cloud-on-debian-wheezy/
* https://www.inkandswitch.com/local-first.html
* https://www.rechberger.io/tutorial-how-to-build-your-own-server-infrastructure-using-ansible/


### Web GUI automated run your own cloud systems
Prefer turnkey web gui based management?  Take a look at these.
* https://caprover.com/
* https://cloudron.io/
* https://cozy.io/
* https://nextcloud.com/
* https://owncloud.org/
* https://sandstorm.io/
* https://yunohost.org/

### Useful tools and services
* https://argoproj.github.io/projects/argo-cd
* https://www.ansible.com/overview/how-ansible-works
* https://www.backblaze.com/
* https://github.com/sshuttle/sshuttle
* https://grafana.com/
* https://github.com/longhorn/longhorn
* https://microk8s.io/
* http://play-with-docker.com/
* https://ngrok.com/
* https://openebs.io/
* https://www.portainer.io/
* https://prometheus.io/
* https://traefik.io/
* https://www.zerotier.com/

### How to support this project
* contribute feedback/ideas/recommendations/discoveries
* report issues (this project contains moving targets)
* create online videos
* share on social media
* sponsor

Have a suggestion, question, comment, or request?  Submit a [new issue](https://github.com/technomada/cloud-from-scratch/issues/new).

Contact: cfs2006@textyio.com

