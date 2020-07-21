## Using Subdomains


Assuming you're running a docker container called nextcloud on your local node and want to map https://nextcloud.example.com into it...

On your **EDGE NODE**

Edit Caddyfile
```
$ vim ~/Caddyfile

nextcloud.example.com {
	tls cert@example.com

	reverse_proxy 10.1.1.2:80
	}
```

On your **LOCAL NODE**

Edit Caddyfile
```
nextcloud.domain.com:80 {

	reverse_proxy nextcloud:80
	}
```

Restart caddy on your edge and local nodes.

note: Caddy supports dynamic "on_demand" sub domain certs.. like this...
```
*.example.com {
	tls cert@example.com {
		on_demand
		}
	}
```


