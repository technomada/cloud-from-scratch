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
