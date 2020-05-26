# Cloud From Scratch
Build your own personal cloud system from scratch.

You may be surpized to find that building and maintaining your own cloud is not only not difficult, but really fun and rewarding. The following instructions should provide everything you need to build a multi host capable personal cloud computing platform from scratch. Using primarily existing technologies and inspired by the "Arch Way" (Simplicity, Modernity, Pragmatism, User centrality, Versatility) this cloud design project aims to require a small amount of technical attention and allow a wide range of functionality. After completing the following steps you'll have yourself a fully functional personal cloud system ready to be shaped into the services you wish. I don't know about you, but I know I'm excited, so let's get started.


# Overview
When completed the system looks something like this.

```
[ EDGE NODE VPS ]                     |               [ HOME SERVER ]
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
