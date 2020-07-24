
https://docs.docker.com/engine/install/debian/

## Install Docker

[tested: 2020.07.24]

Location: Follow these instructions on your **EDGE NODE**

```
$ apt install gnupg2 software-properties-common
$ curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
$ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
$ apt update
$ apt install docker-ce docker-ce-cli containerd.io
```

Create a docker network called `master`
```
$ docker network create -d bridge master
```


Test (this should download and run docker hello world image)
```
$ docker run hello-world  
```
