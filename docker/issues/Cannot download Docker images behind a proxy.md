# docker 无法获取镜像


## 问题描述：
I installed docker on my ubuntu 13.10 and when I type in my console:

````ssh
sudo docker pull busybox
````

I get the following error:

````ssh
Pulling repository busybox
2014/04/16 09:37:07 Get https://index.docker.io/v1/repositories/busybox/images: dial tcp: lookup index.docker.io on 127.0.1.1:53: no answer from server
Docker version:
````

## 处理方式：
[Cannot download Docker images behind a proxy](https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy)

That's an old subject, but I'll give my two cents in case someone else is looking for advice.

Here is a link to the official docker doc for proxy http: https://docs.docker.com/engine/admin/systemd/#http-proxy

A quick outline:

- First, create a systemd drop-in directory for the docker service:

````ssh
mkdir /etc/systemd/system/docker.service.d
````

- Now create a file called /etc/systemd/system/docker.service.d/http-proxy.conf that adds the HTTP_PROXY environment variable:

````ssh
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80/"
````

- If you have internal Docker registries that you need to contact without proxying you can specify them via the NO_PROXY environment variable:

````ssh
Environment="HTTP_PROXY=http://proxy.example.com:80/"
Environment="NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com"
````

- Flush changes:

````ssh
$ sudo systemctl daemon-reload
````
- Verify that the configuration has been loaded:

````ssh
$ sudo systemctl show --property Environment docker
Environment=HTTP_PROXY=http://proxy.example.com:80/
````

- Restart Docker:

````ssh
$ sudo systemctl restart docker
````
