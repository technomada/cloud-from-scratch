# GOGS (incomplete)
A painless self-hosted Git service

```
$ mkdir ~/gogs
$ vim ~/Caddyfile

https://example.com {

# add this section for gogs
	route /gogs* {
		redir /gogs /gogs/
		uri strip_prefix /gogs
		reverse_proxy gogs:3000
		}
	
  }
```

Start server and make sure folders are pi user 
```
$ sudo docker run -it --rm --name=gogs --network=master -p 10022:22 -p 3000:3000 -v /home/pi/gogs:data gogs/gogs-rpi
$ sudo chown pi:pi ~/gogs
```

Browse [http://example.com:3000/]  to configure gogs
- Database Type: SQLLite3
- Application URL: https://example.com/gogs/

 Add EXTERNAL_URL setting in config file.
```
[server]
EXTERNAL_URL     = http://example.com/gogs/

[database]
PATH     = /data/gogs/data/gogs.db
```

Browse [http://example.com/gogs/]

----------------------
https://github.com/gogs/gogs/tree/master/docker
