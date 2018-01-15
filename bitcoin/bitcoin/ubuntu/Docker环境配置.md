# Docker  环境安装

（注：阅读该篇文章前，务必配置好Ubuntu基础环境）

安装依赖包

```
sudo apt-get install linux-image-generic-lts-trusty
# 安装 docker.io
sudo apt-get install docker.io
```

history

```
    1  sudo
    2  sudo ls
    3  ls
    4  useradd docker -g docker
    5  sudo apt-get update
    7  sudo apt-get install linux-image-generic-lts-trusty
    8  ls
    9  sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
   10  sudo vim /etc/apt/sources.list
   11  sudo apt-get install vim
   12  sudo apt-get update 
   13  vim test.md
   14  ls
   15  rm -f test.md 
   16  ls
   17  sudo vim /etc/apt/sources.list
   18  sudo apt-get update 
   19  sudo apt-get install linux-image-generic-lts-trusty
   20  sudo vim /etc/apt/sources.list
   22  free -m
   24  reboot
   25  free -m
   26  vim /etc/apt/sources.list
   27  sudo vim /etc/apt/sources.list
   28  sudo apt-get update
   29  ls
   30  sudo apt-get install linux-image-generic-lts-trusty
   31  vim /etc/apt/sources.list
   32  sudo vim /etc/apt/sources.list
   33  ls
   36  sudo apt-get update 
   37  sudo apt-get install linux-image-generic-lts-trusty
   38  useradd docker-g docker
   39  docker
   42  whereis vim
   43  whereis docker
   44  ls
   45  sudo apt-get install wget
   46  wget -qO-https://get.docker.com/ | sh
   47  wget -qO- https://get.docker.com/ | sh
   48  sudo reboot
   49  ls
   50  sudo docker ps
   51  history
   52  history > docker
```

下载docker所需镜像

```
# 下载官网镜像
sudo docker pull ubuntu:14.04
# 下载最新的官方镜像
sudo docker pull ubuntu:latest
```

若下载失败,自行登陆官网下载

```
# 加载镜像
sudo docker load -i ubuntu_14.04.tar
# 查看当前镜像
sudo docker images
# 制作并运行容器
# --name: 指定容器名称
# run -v: 将主机目录映射到容器目录
# ubuntu:latest: 镜像名与Tag
sudo docker run -it --name bitcoin -v /home/cly/Desktop/dockerbtc:/cloud/btc ubuntu:latest /bin/bash
# 运行容器
docker exec -it bitcoin /bin/bash
```



