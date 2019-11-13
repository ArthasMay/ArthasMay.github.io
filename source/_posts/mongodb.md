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



