# Bitcoin 交叉编译Window客户端

（注：阅读该篇文章前，务必配置好Bitcoin基础环境）

## 启动Docker

```
# 查看当前容器
sudo docker ps -a
# 启动容器
sudo docker start bitcoin
# 运行容器(CONTAINER)
# bitcoin: 可以填写容器名或者容器ID
sudo docker exec -it bitcoin /bin/bash
```

## 基础环境配置

安装 基础环境

```
# 更新云信息
apt-get update
# 下载 git
apt-get install git
# 安装 autogen
apt-get install autoconf
# 安装编译环境
apt-get install gcc
apt-get install g++
```

初始化目录

```
mkdir -p bitcoin/bitcoin/win
cd bitcoin/bitcoin/win
```

克隆Bitcoin项目

```
# 克隆最新版 bitcoin 项目
git clone https://github.com/bitcoin/bitcoin.git
# 进入bitcoin项目
cd bitcoin
```

## 基础依赖库配置

```
apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils python3
apt-get install software-properties-common
add-apt-repository ppa:bitcoin/bitcoin
apt-get update
apt-get install libdb4.8-dev libdb4.8++-dev
apt-get install libminiupnpc-dev
apt-get install libzmq3-dev
```

```
# 程序所需的 boost 库安装包
apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev
# 若执行不顺利可以安装整个 Boost 库
apt-get install libboost-all-dev
```

若需要Qt界面可以安装以下库

```
# Qt 5 基础库
apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
# Qt 二维码扩展库
apt-get install libqrencode-dev
```

## 编译客户端

```
# 生成安装脚本
./autogen.sh
# 安装检测环境
./configure
# 构建项目
make
```



