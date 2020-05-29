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
