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

安装 Git

```
# 更新云信息
apt-get update
# 下载 gitV
apt-get install git
# 安装 autogen
apt-get install autogen
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

安装 32-bit MinGw 跨平台构建程序

MinGw 是跨平台的编译器, 我们这里欲生成 32 bit 的 bitcoin 客户端, 故我们选择了 i686 版本

```
apt install g++-mingw-w64-i686 mingw-w64-i686-dev
```



