## Local node Raspbian Caddy

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
$ mkdir -p ~/www/example.com
```
(replace `example.com` with your domain name)

Start caddy
```
$ sudo docker run -d --restart=always --name=caddy_web_server -p 80:80 -p 443:443 --network=master -v /home/pi/Caddyfile:/etc/caddy/Caddyfile -v /home/pi/www:/usr/share/caddy -v /home/pi/certs:/data caddy
```

Testing
```
$ sudo vim /etc/hosts
```
add
```
127.0.0.1 example.com
```
.. replace  `example.com` with the domain you're using.

```
$ curl -v example.com
```
You should see.. Yay!  It Really Works! 

if no, try learning more with...
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
or better...
```
$ docker exec -it caddy_web_server caddy reload --config /etc/caddy/Caddyfile
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

### Debugging
If you don't see what you expect you can debug by reactivating the respond section (and deactivating the reverse_proxy) of the Caddyfile on the local node or remote node and restarting caddy and use your hosts file to bypass Cloudflare.  You can also test wireguard connections with wg and pinging.

The edge server should be able to ping the local node and the local node should be able to ping the edge server.

eg
```
root@edge:~# ping 10.1.1.2

PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=107 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=63.1 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=62.8 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=69.8 ms
```

```
pi@raspberrypi:~ $ ping 10.1.1.1

PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=62.5 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=64 time=79.5 ms
64 bytes from 10.1.1.1: icmp_seq=3 ttl=64 time=63.1 ms
64 bytes from 10.1.1.1: icmp_seq=4 ttl=64 time=65.8 ms
```
