[Update Version 2020.05.25]

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


```
Edit sources.list and add buster-backports

$ vim /etc/apt/sources.list.d/sources.list
  deb https://deb.debian.org/debian buster-backports main

$ apt update
$ apt -t buster-backports install "wireguard"

$ reboot
```

