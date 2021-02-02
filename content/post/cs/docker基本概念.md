---
title: "docker基本概念"
date: 2018-06-26T14:08:52+08:00
tags: ["docker"]
categories: ["CS"]
toc: true
---
# docker
安装ubuntu自带的docker:
```zsh
sudo apt install docker-io docker-compose
sudo systemctl start docker
sudo systemctl enable docker
```
安装最新的docker:
```zsh
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get -y install docker-ce
sudo usermod -aG docker $USER
```

## bootfs
## rootfs
换源：
```zsh
vi /etc/docker/daemon.json 

"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"],
```
重启：
```zsh
sudo systemctl daemon-reload
sudo systemctl restart docker
```
安装docker-compose:
```zsh
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```
国内源以前有下面这些，但现在要么要认证，要么关了：
* Azure 中国镜像 https://dockerhub.azk8s.cn
* 阿里云加速器 (需要登录阿里云账户)
* 七牛云加速器 https://reg-mirror.qiniu.com
* 中科大镜像源
* DaoCloud 加速器

## image
```zsh
docker images
docker rmi -f image
docker tag image image:tag
docker save -o image.tar image
docker load -i image.tar
```

## container
```zsh
docker run -id --name=c_name -p 80:80 -v /volumes:/etc/volumes image
docker exec -it c_name bash
docker run -it -p 80:80 -v /volumes:/etc/volumes image bash
docker ps -a
docker rm -f c_name

docker inspect c_name
docker commit -a "author" -m "message" c_name image
```
## remote
```zsh
docker pull image
docker push image
docker search image
```
### private

## compose
```zsh
docker-compose up -d
docker-compose up -d --force-recreate --no-deps
```

```config
'proxy' => '172.16.0.200:1080', 
'proxyuserpwd' => 'user:pass',
'trusted_proxies' => array('172.16.0.200'),
```