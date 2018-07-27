---
title: docker私有仓库的建立
date: 2018-05-16 15:53:54
tags: docker
---


### 使用官方提供的registry镜像，可以达到建立私有镜像库的目的
#### 启动服务
``` shell
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

#### 假设本地已有python:2镜像,首先需修改image tag
``` shell
docker tag python:2 localhost:5000/python
```

#### 上传镜像
``` shell
docker push localhost:5000/python
```

#### 拉取镜像
``` shell
docker pull localhost:5000/python
```

#### 设置服务密码

* 设置密码
``` shell
mkdir auth
docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
```

* 重新启动服务
``` shell
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```

* 登录
``` shell
docker login myregistrydomain:5000
```

#### 使用docker-compose
``` shell
registry:
  restart: always
  image: registry:2
  ports:
    - 5000:5000
  environment:
    REGISTRY_AUTH: htpasswd
    REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
  volumes:
    - /path/data:/var/lib/registry
    - /path/auth:/auth
```

### 远程私有库 
对于非本地私有库，docker默认只支持https连接，或者修改本地机器的设置,在/etc/docker/daemon.json中增加
``` python
{"incecure-registries": ["myregistry:example.com:5000"]}
```
然后重启docker服务


### 参考:
* https://docs.docker.com/registry/deploying/#stop-a-local-registry
* https://github.com/docker/distribution/issues/1874
