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
