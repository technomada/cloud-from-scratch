
https://docs.docker.com/engine/install/debian/

##Install Docker

Location: Follow these instructions on your **EDGE NODE**

```
$ apt install gnupg2 software-properties-common
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
$ apt update
$ apt install docker-ce docker-ce-cli containerd.io
```

Test (this should download and run docker hello world image)
```
$ docker run hello-world  
```
