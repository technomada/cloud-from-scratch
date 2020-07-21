## Using Subdomains

In Edge Node
```
nextcloud.domain.com {
	tls cert@domain.com

	reverse_proxy 10.10.10.2:80
	}
```

In Local Node
```
nextcloud.domain.com:80 {

	reverse_proxy nextcloud:80
	}
```

note: Caddy supports dynamic "on_demand" sub domian certs.. like this...
```
*.domain.com {
	tls cert@domain.com {
		on_demand
		}
	}
```


