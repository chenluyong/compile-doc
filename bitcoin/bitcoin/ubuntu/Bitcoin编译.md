# Bitcoin 交叉编译

（注：阅读该篇文章前，务必配置好Bitcoin基础环境）

## 启动Docker

```
# 运行容器
docker exec -it bitcoin /bin/bash
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

克隆bitcoin项目

```
# 克隆最新版 bitcoin 项目
git clone https://github.com/bitcoin/bitcoin.git
```



