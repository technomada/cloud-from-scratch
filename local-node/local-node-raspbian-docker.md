```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

Verify
```
$ docker version
$ sudo docker info
$ sudo docker run hello-world
```


Create Network "Master"
```
$ sudo docker network create -d bridge master
```
