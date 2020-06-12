[ Modified 2020.06.10 ]

# Express
A simple static file / express api based generic application setup.

Make example app directories
```
$ mkdir -p ~/app/example/www
$ cd ~/app/example
```

Edit index.html
```
$ vim www/index.html
```
```
Hello!
```

Edit server.js
```
$ vim server.js
```

Create simple express application
```
let express = require('express')
let path = require('path')

let app = express()
app.use(express.static(path.join(__dirname,'www')))

app.all('/api',(req,res)=>{
	res.json(true)
	})

app.listen(3000,()=>{})
```

Init app
```
$ sudo docker run -it --rm -v $PWD:/usr/src/app -w /usr/src/app node npm init
$ sudo docker run -it --rm -v $PWD:/usr/src/app -w /usr/src/app node npm i -S express
```

Start app
```
$ sudo docker run -d --restart=always --name=example_app --network=master -v /home/pi/apps/example:/usr/src/app -w /usr/src/app node node server.js
```

Map app into your domain
```
$ cd ~
$ vim Caddyfile
```

Add this route to your domain
```
http://example.com {

	route /example* {
		redir /example /example/
		uri strip_prefix /example
		reverse_proxy example_app:3000
		}
}
```

Restart Caddy
```
$ sudo docker restart caddy_web_server
```

browse 	[http://example.com/example]  --- Hello!

browse	[http://example.com/example/api] --- true
