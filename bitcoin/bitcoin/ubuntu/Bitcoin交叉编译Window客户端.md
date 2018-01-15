# Bitcoin 交叉编译Window客户端

（注：阅读该篇文章前，务必配置好Bitcoin基础环境）

## 启动Docker

```
# 查看当前容器
sudo docker ps -a
# 运行容器
# bitcoin: 可以填写容器名或者容器ID
sudo docker exec -it bitcoin /bin/bash
```

## 基础环境配置

安装 Git

```
# 更新云信息
apt-get update
# 下载 git
apt-get install git
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
```



