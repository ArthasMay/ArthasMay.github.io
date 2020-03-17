---
title: MongoDB记录
date: 2019-11-08 11:11:12
tags: 
- MongoDB
---

## Mac 搭建MongoDB的开发环境

### brew cask 安装

``` bash
# 由于MongoDB已经不再开源，只能通过brew cask安装社区版本
$ brew tap mongodb/brew
$ brew install mongodb-community
```
默认创建相关的接口
a configuration file: /usr/local/etc/mongod.conf
a log directory path: /usr/local/var/log/mongodb
a data directory path: /usr/local/var/mongodb

### 配置通过CommandLineTool

``` sh
# .zshrc or .bash_profile 配置文件
export MONGOPATH=/usr/local/Cellar/mongodb-community/4.2.1/bin
export PATH=$PATH:$MONGOPATH
```

### 配置命令

``` sh
# 开启MongoDB
$ mongod 
# 48错误码 /tmp/mongodb-27017.sock没有权限，解决方法：直接删除
# 100错误码 /data/db 文件夹没有写权限，解决方法：chmod a+w xxx（需要深入了解linux/unix文件权限了）

# 打开mongo repl
$ monngo
```

### mongo的命令
``` 
// 退出mongo
use admin;
db.shutdownServer();


```





