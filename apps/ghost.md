[updated 2020.06.12]

# Ghost
Ghost is a free and open source blogging platform written in JavaScript and distributed under the MIT License, designed to simplify the process of online publishing for individual bloggers as well as online publications.



```
$ mkdir ~/ghost
$ vim ~/Caddyfile
```
```
http://example.com {
      
      
        # add route
        route /blog* {
                redir /blog /blog/
                reverse_proxy ghost_blog:2368
                }
        }
```

Reload caddy config
```
$ sudo docker exec caddy_web_server caddy reload --config /etc/caddy/Caddyfile
```
(replace example.com with your domain)

```
$ sudo docker run -d --restart=always --name ghost_blog --network master -e url=http://example.com/blog/ -v /home/pi/ghost:/var/lib/ghost/content ghost
```

Check that it's working
```
$ sudo docker logs -f ghost_blog
```
ctrl+c to return to term

Browse [ https://example.com/blog/ghost/ ] to setup your ghost install.

Visit [ https://example.com/blog/ ] to visit your blog.

### related links
* https://hub.docker.com/_/ghost | https://ghost.org/docs/concepts/config/
* https://github.com/docker-library/ghost/issues/120 --- email config
