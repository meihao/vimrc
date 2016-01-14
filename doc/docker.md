Dodcker manual
==============

#Basic Config

###Install on different system
-----------
docker hosts
54.234.135.251  get.docker.io
54.234.135.251  cdn-registry-1.docker.io
```
#Debian8
need kernel > 3.8, add the below content into sources.list
#deb http://http.debian.net/debian jessie-backports main
#apt-get install docker.io
有docker 的用户组, 为了避免使用sudo, 需要把当前用户加入到docker组,
#sudo usermod -aG docker your_username

remove docker
#apt-get purge docker-io
or
#apt-get autoremove --purge docker-io
#rm -rf /var/lib/docker

#Centos7
sudo yum install docker
sudo systemctl start docker
sudo systemctl enable docker
```

###Verify Installation
-----------
```
docker version
Client version: 1.6.2
Client API version: 1.18
Go version (client): go1.3.3
Git commit (client): 7c8fca2
OS/Arch (client): linux/amd64
Server version: 1.6.2
Server API version: 1.18
Go version (server): go1.3.3
Git commit (server): 7c8fca2
OS/Arch (server): linux/amd64
```

###Docs
-----------
```
#docker info
#docker search xxxx 搜索相关的官方docker镜像(index.docker.io)
#docker pull xxxx 下载相关的镜像,    用户名/镜像名, 
                (根据 /var/lib/docker/repositories-aufs 存储在/var/lib/docker/graph/
                 /var/lib/docker/aufs/diff 里面存在的镜像)
e.g. docker run user/project_image command 
#docker run xxxx echo "hello world"  运行xxxx镜像, 并且在镜像中运行echo 命令

docker 无法实现交互模式, 也就是比如apt-get install xxx 需要增加-y 参数默认

当你对某一个容器做了修改之后（通过在容器中运行某一个命令），可以把对容器的修改保存下来，
这样下次可以从保存后的最新状态运行该容器。
docker中保存状态的过程称之为committing，它保存的新旧状态之间的区别，从而产生一个新的版本。

首先使用docker ps -l命令获得安装完ping命令之后容器的id。然后把这个镜像保存为learn/ping。
$sudo docker ps -l
CONTAINER ID        IMAGE                   COMMAND                CREATED             STATUS                     PORTS               NAMES
66a36c7e1bfc        learn/tutorial:latest   "apt-get install -y    8 minutes ago       Exited (0) 8 minutes ago                       nostalgic_euclid 

运行docker commit，可以查看该命令的参数列表。
2. 你需要指定要提交保存容器的ID。(译者按：通过docker ps -l 命令获得)
3. 无需拷贝完整的id，通常来讲最开始的三至四个字母即可区分。（译者按：非常类似git里面的版本号)

$sudo docker commit 66a learn/ping
执行完docker commit命令之后，会返回新版本镜像的id号。

一定要使用新的镜像名learn/ping来运行ping命令。(译者按：最开始下载的learn/tutorial镜像中是没有ping命令的)

% sudo docker run learn/tutorial ping www.baidu.com
exec: "ping": executable file not found in $PATH
FATA[0000] Error response from daemon: Cannot start container 23af03a7c2e1692b0474437d16165390735cc29ba49734261902de8ba7d6d030: [8] System error: exec: "ping": executable file not found in $PATH 

% sudo docker run learn/ping ping www.baidu.com
PING www.a.shifen.com (220.181.112.244) 56(84) bytes of data.
64 bytes from 220.181.112.244: icmp_req=1 ttl=53 time=38.1 ms
64 bytes from 220.181.112.244: icmp_req=2 ttl=53 time=37.5 ms
64 bytes from 220.181.112.244: icmp_req=3 ttl=53 time=38.0 ms


使用docker ps命令可以查看所有正在运行中的容器列表，
$sudo docker ps -l

使用docker inspect命令我们可以查看更详细的关于某一个容器的信息。
$sudo docker inspect 20f66
$sudo docker inspect 20f66 -f
$sudo docker inspect --format='{{.State.Running }}' 20f66 来限定输出

1. docker images命令可以列出所有安装过的镜像。
2. docker push命令可以将某一个镜像发布到官方网站。
3. 你只能将镜像发布到自己的空间下面。这个模拟器登录的是learn帐号。


$sudo docker run -i -t user/project /bin/bash 进入docker的交互式shell
    -i 标志保证容器中STDIN是开启的. -t表示告诉docker要为创建的容器分配一个伪tty终端
    --name alias_name 可以为这个docker指定一个别名
$sudo docker rm alias_name 来删除同名容器
#exit 退出该docker

$sudo docker start alias_name 来启动指定的docker容器, restart来重启
也可以使用attach来重连该docker容器 
$sudo docker start 23af03a
Error response from daemon: Cannot start container 23af03a: [8] System error: exec: "ping": executable file not found in $PATH
FATA[0000] Error: failed to start one or more containers 
$sudo docker start 10ae45
10ae45
$sudo docker attach 10ae45
64 bytes from 220.181.112.244: icmp_req=9 ttl=53 time=38.4 ms
64 bytes from 220.181.112.244: icmp_req=10 ttl=53 time=37.5 ms
64 bytes from 220.181.112.244: icmp_req=11 ttl=53 time=38.1 ms
64 bytes from 220.181.112.244: icmp_req=12 ttl=53 time=37.6 ms
^C
--- www.a.shifen.com ping statistics ---
12 packets transmitted, 12 received, 0% packet loss, time 11013ms
rtt min/avg/max/mdev = 37.561/38.483/44.528/1.886 ms


进入docker伪tty后, 可以像正常操作linux一样操作docker

docker ps -a 列出所有docker容器.



创建守护式容器(deamonized container) 而不是(交互式运行容器)interactive container
$sudo docker run --name daemon_dave -d ubuntu /bin/sh -c "while true;do echo hello world; sleep 1; done"
    -d  参数会把容器放到后台运行

$sudo docker logs daemon_dave 来查看日志
$sudo docker logs -f daemon_dave 类似tail -f
$sudo docker logs --tail 10 daemon_dave 获取日志的最后10行
$sudo docker logs --tail 0 -f daemon_dave 来跟踪某个容器的最新日志.
$sudo docker top daemon_dave  查看容器内部运行的进程


在容器内部运行进程
$sudo docker exec -d daemon_dave touch /etc/new_config_file 
$sudo docker exec -t -i daemon_dave /bin/bash

停止守护式容器
$sudo docker stop daemon_dave

查看最后x个容器
docker ps -n x

自动重启容器, always/on-failure:5 可选的重启次数,on-failure只有当容器的退出代码为非0值的时候才会自动重启, 可以接数字来限定重启次数
$sudo docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c "while true;do echo hello world; sleep 1; done"
    --restart=always
    --restart=on-failure
    --restart=on-failure:2

$sudo docker rm `docker ps -a -q` 一次性删除所有容器


列出镜像
$sudo docker images

$sudo docker run -t -i --name new_container ubuntu:12.04 /bin/bash 通过指定版本号来拉取指定的镜像

搜索镜像
$sudo docker search debian

创建docker hub帐号
https://hub.docker.com/account/signup
测试帐号
$sudo docker login
Username:xxxxx
Password: 
Email: xxxx@gmail.com
WARNING: login credentials saved in /root/.dockercfg.
Login Succeede


对镜像作了修改后, 可以commit, 比如安装软件
$sudo docker commit 3aab3ce  user_name/project

$sudo docker commit dffb66497 rainysia/learn
b7f12b9b4b29a0d18ccb8e2490aa0fb4eece919ddfe59a712171a90ac963938c

$sudo docker commit -m="A new custom image for test base on ubuntu12.04" --author="rainysia" dffb66497 rainysia/learn:test
    最后一个是test是tag,类似alias_name别名
查看该镜像的详细
$sudo docker inspect rainysia/learn:test    


运用dockerfile来构建镜像
dockerfile
----------------------------
# Version: 0.0.1
FROM debian                      # FROM来指定一个已经存在的镜像, 后续指令都是基于这个镜像
MAINTAINER rainysia "rainysia@gmail.com"
RUN apt-get update                          # run指定会默认shell用/bin/sh -c来执行.如果不希望或者不支持shell平台.可以用exec方式   RUN ["apt-get", " install", "-y", "nginx"]
RUN apt-get install -y nginx
RUN echo 'Hi, i am in your container' \
        > /usr/share/nginx/html/index.html
EXPOSE 80                                   #指定打开80端口
----------------------------
执行docker build来build一个镜像
$sudo docker build -t="rainysia/static_web" .

------------------------------------------------
sudo docker build -t="rainysia/static_web" .
Sending build context to Docker daemon 2.048 kB
Sending build context to Docker daemon 
Step 0 : FROM learn/ping:latest
 ---> 9e8823ddedf9
Step 1 : MAINTAINER rainysia "rainysia@gmail.com"
 ---> Running in 5e6fc8254653
 ---> 3077170bae60
Removing intermediate container 5e6fc8254653
Step 2 : RUN apt-get update
 ---> Running in 2026e6c913cf
Ign http://archive.ubuntu.com precise InRelease
Hit http://archive.ubuntu.com precise Release.gpg
Hit http://archive.ubuntu.com precise Release
Hit http://archive.ubuntu.com precise/main amd64 Packages
Get:1 http://archive.ubuntu.com precise/main i386 Packages [1641 kB]
Get:2 http://archive.ubuntu.com precise/main TranslationIndex [3706 B]
Get:3 http://archive.ubuntu.com precise/main Translation-en [893 kB]
....
------------------------------------------------

也可以为镜像设置tag
$sudo docker build -t="rainysia/static_web:tag_v1" .
也可以指定dockerfile
$sudo docker build -t="rainysia/static_web:tag_v1" \
                       git@github.com:rainysia/docker-static_web 

docker 构建文件支持.dockerignore文件来忽视对应的文件, 类似.gitignore
docker 在build失败前, 会在每步生成一个id, 可以基于这个id, 来继续最后一个成功后的步骤.
$sudo docker run -t -i xxxeeeeet294 /bin/bash


------------------------------------------------
$sudo docker build -t="rainysia/static_web" .                                                                                                                                      [15:37:38]
Sending build context to Docker daemon 2.048 kB
Sending build context to Docker daemon 
Step 0 : FROM debian
latest: Pulling from debian
6d1ae97ee388: Pull complete 
8b9a99209d5c: Pull complete 
debian:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:2a5204f0a00510e9b99d7ec0fe40bea495b8e3630b820b04675488aad4188e06
Status: Downloaded newer image for debian:latest
 ---> 8b9a99209d5c
Step 1 : MAINTAINER rainysia "rainysia@gmail.com"
 ---> Running in 3de883382215
 ---> 491af30c07dc
Removing intermediate container 3de883382215
Step 2 : RUN apt-get update
 ---> Running in d3979fb6827a
Get:1 http://security.debian.org jessie/updates InRelease [63.1 kB]
Get:2 http://security.debian.org jessie/updates/main amd64 Packages [217 kB]
Ign http://httpredir.debian.org jessie InRelease
Get:3 http://httpredir.debian.org jessie-updates InRelease [136 kB]
Get:4 http://httpredir.debian.org jessie Release.gpg [2373 B]
Get:5 http://httpredir.debian.org jessie Release [148 kB]
Get:6 http://httpredir.debian.org jessie-updates/main amd64 Packages [3619 B]
Get:7 http://httpredir.debian.org jessie/main amd64 Packages [9035 kB]
Fetched 9606 kB in 32s (293 kB/s)
Reading package lists...
 ---> 488df59d3a97
Removing intermediate container d3979fb6827a
Step 3 : RUN apt-get install -y nginx
 ---> Running in caee197a17aa
Reading package lists...
Building dependency tree...
Reading state information...
The following extra packages will be installed:
  fontconfig-config fonts-dejavu-core geoip-database init-system-helpers
  .........
Setting up nginx-full (1.6.2-5) ...
invoke-rc.d: policy-rc.d denied execution of start.
Setting up nginx (1.6.2-5) ...
Setting up rename (0.20-3) ...
update-alternatives: using /usr/bin/file-rename to provide /usr/bin/rename (rename) in auto mode
Setting up xml-core (0.13+nmu2) ...
Processing triggers for libc-bin (2.19-18+deb8u1) ...
Processing triggers for systemd (215-17+deb8u2) ...
Processing triggers for sgml-base (1.26+nmu4) ...
 ---> a74b1ee93ca8
Removing intermediate container caee197a17aa
Step 4 : RUN echo 'Hi, i am in your container'         > /usr/share/nginx/html/index.html
 ---> Running in d791b08c3fdd
 ---> 75a3f6d8fd40
Removing intermediate container d791b08c3fdd
Step 5 : EXPOSE 80
 ---> Running in cd89bc68a838
 ---> 9987527a0059
Removing intermediate container cd89bc68a838
Successfully built 9987527a0059
------------------------------------------------

查看build的镜像
$sudo docker images
======================================================= 
$sudo docker images
REPOSITORY            TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
rainysia/static_web   latest              9987527a0059        19 minutes ago       196.2 MB
<none>                <none>              30d0c78c67c9        31 minutes ago       168.8 MB
rainysia/learn        test                7dee5de20590        43 minutes ago       139.9 MB
rainysia/learn        latest              b7f12b9b4b29        44 minutes ago       139.9 MB
learn/ping            latest              9e8823ddedf9        24 hours ago         139.9 MB
debian                latest              8b9a99209d5c        2 weeks ago          125.1 MB
learn/tutorial        latest              8dbd9e392a96        2.697135 years ago   128 MB
=======================================================
$sudo docker history 9987527a0059 来查看image的历史操作

基于新构建的镜像启动一个新容器.
$sudo docker run -d -p 80 --name static_web rainysia/static_web nginx -g "daemon off;"
0daff8960db967d4bba0ac4851a17862742b15350a4662582133bacb5ae82192

$sudo docker ps -l 查看端口情况
CONTAINER ID        IMAGE                        COMMAND                CREATED              STATUS              PORTS                   NAMES
0daff8960db9        rainysia/static_web:latest   "nginx -g 'daemon of   About a minute ago   Up About a minute   0.0.0.0:32768->80/tcp   static_web     
容器中的80端口被映射到了宿主机的32768端口
也可以通过docker port 来查看容器的端口占用情况
$sudo docker port 0daff8960db9 80
0.0.0.0:32768
    -p 选项可以指定映射端口
$sudo docker run -d -p 80:80 --name static_web rainysia/static_web \
        nginx -g "daemon off;"
可以通过 http://127.0.0.1:32768/ 来查看映射
    -p 127.0.0.1:80:80 把80端口绑定到宿主机的127.0.0.1:80
    -P 来公开在dockerfile里面定义的expose的所有端口.
$sudo docker run -d -P --name static_web rainysia/static_web nginx -g "daemon off;"
$sudo docker run -d -p 192.168.85.123:33333:80 --name static_web rainysia/static_web nginx -g "daemon off;"

更新docker images里面的内容后. 
需要stop掉当前的docker, 然后rm 掉, 启新的
$sudo docker ps -a 
$sudo docker stop xxxxid
$sudo docker rm xxxxid
$sudo docker start new_xxxxid

$sudo docker rmi image_id 根据image_id删除镜像


Dockerfile 的命令
================
CMD 指定一个容器启动时要运行的命令, 类似RUN, RUN是指定镜像被构建时要运行的命令.
    CMD ["/bin/true"]
    CMD ["/bin/true", "-1"]

ENTRYPOINT 提供的命令不容易在启动容器的时候被覆盖
    ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]

WORKDIR 用来在镜像创建一个新容器时,在容器内部设置一个工作目录, ENTRYPOINT/CMD 指定的程序会在这目录下运行
    WORKDIR /opt/webapp/db
    RUN bundle install
    WORKDIR /opt/webapp
    ENTRYPOINT ["rackup"]
    可以通过-w来覆盖工作目录
    docker run -ti -w /var/lig ubuntu pwd

ENV 在镜像构建过程中设置环境变量
    ENV RVM_PATH /home/rvm
    类似RUN gem install unicorn

USER 指定该镜像会以什么用户来运行,不指定默认root
    USER nginx
    USER uid:gid
    USER user:group
    USER user:gid
    USER uid:group

VOLUME 向容器添加卷
    VOLUME ["/opt/project"]
    VOLUME ["/opt/project", "/data"]

ADD 将构建环境下的文件和目录复制到镜像中 ADD host_file dest_file, host_file支持url地址
    ADD software.inc /opt/application/software.inc

COPY ,不存在会创建目录
    COPY /etc/nginx/  /etc/apache2/nginx

ONBUILD 为镜像添加触发器, docker inspect 可以看见OnBuild部分
    ONBUILD ADD . /app/src
    ONBUILD RUN cd /app/src && make

==================
$sudo docker push user/alias_name push到docker hub上
$sudo docker push rainysia/static_web
The push refers to a repository [rainysia/static_web] (len: 1)
9987527a0059: Image already exists 
75a3f6d8fd40: Image successfully pushed 
a74b1ee93ca8: Image successfully pushed 
488df59d3a97: Image successfully pushed 
491af30c07dc: Image successfully pushed 
8b9a99209d5c: Image successfully pushed 
6d1ae97ee388: Image successfully pushed 
Digest: sha256:c6ea3b8f3a01dbac27945c576e41c5069adf6aaaa2a7c7d91141437359768edc

为镜像打标签
$sudo docker tag xxxxxxid docker.example.com:5000/rainysia/static_web
然后快速部署
$sudo docker run -t -i docker.example.com:5000/rainysia/static_web /bin/bash


==============================
修改dockerfile

$sudo docker iamges
$sudo docker build -t rainysia/new_project_name .
$sudo docker rm `docker ps -q -a`
$sudo docker history rainysia/nginx
$sudo docker push rainysia/new_project_name

$sudo docker run -d -p 127.0.0.1:44444:80 --name website \
        -v $PWD/website:/var/www/html/website \
        rainysia/nginx nginx

sudo docker run -d -p 127.0.0.1:44444:80 --name test_web rainysia/nginx nginx

sudo docker logs -f test_web 查看docker运行的日志
sudo docker port test_web 80 查看docker容器的端口映射
sudo docker top test_web  查看docker容器的正在运行的程序

=======================================
卷可以在容器间共享, 即使容器停止. 卷里的内容依旧存在. 适合对代码作开发和测试
    -v 参数指定了卷的源目录(本地宿主机的目录)和容器里的目的目录. 这两个目录用:分隔,目的目录不存在会自动建
        也可以在目的目录后面加上rw, ro 来指定目的目录的读写状态
$sudo docker run -d -p 80 --name website \
        -v $PWD/website:/var/www/html/website:ro \
        rainysia/new_project_name nginx
    通过卷将$PWD/website 挂载到了容器的/var/www/html/webiste 目录.

$sudo docker run -d -p 127.0.0.1:44444:80 --name test_web \
-v $PWD/website:/var/www/html:ro \
rainysia/nginx nginx

```