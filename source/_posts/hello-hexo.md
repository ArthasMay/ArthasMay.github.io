---
title: Hexo + Next + GHPage/Travis CI 搭建个人技术博客
tags:
- 技术效率
---
## Hexo的官方介绍


Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

### Quick Start

#### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

#### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

#### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

#### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

## 主题（theme）的选择

[NexT站点](http://theme-next.iissnan.com) 推荐主题NexT，至于为什么? 精于心，简于形这条slogan就足够了，相信你也会喜欢上这款简洁的主题的。

## 使用GH(github page) + Travis CI发布

* gh提供稳定的发布hexo的静态站点
* TravisCI简化发布流程，在push之后自动构建

### Guthub上创建博客托管仓库
1. 创建名为xxx.github.io的repo，xxx是自己github的注册名（大小写不敏感）
2. repo准备两条分支(也可以分两个仓库，这里就使用两个branch)
  * master: 托管生成的静态文件
  * hexo: 托管源代码
3. 关联本地hexo仓库和github上远程仓库
``` bash
$ git init 
$ git remote add origin git@github.com:ArthasMay/ArthasMay.github.io.git
$ git pull orign master // 如果远程repo有文件的话，在本地仓库merge
$ git branch -b hexo
$ git add .
$ git commit -m "commit message"
$ git push hexo:hexo
```

### 配置Github Token
那既然需要使用travis自动化更新你的博客，travis自然需要读写你的github上的repo。github提供了token机制来供外部访问你的仓库。
进入[https://github.com/settings/tokens](https://github.com/settings/tokens)，生成一个供travis读写你的github用的token，至于token的权限，选择repo就够了。

### 配置Travis CI

使用github账号登陆travis，如果没有github账号的同学，可以参考《如何提交代码到Github》注册一个自己的github账号。

#### 在Travis进入仓库同步管理

进入[https://travis-ci.org/profile](https://travis-ci.org/profile)，打开刚才托管的hexo博客源码仓库同步开关，不一定是博客网站repo。由于笔者直接把本地hexo博客源码托管到了arthasmay.github.io的hexo分支，所以也就是打开网站repo。如果你不是采取的分支管理，而是将hexo博客源码托管道独立repo，打开对应的github repo开关即可。

#### Travis 配置

* 只在.travis.yml文件存在时编译，必选！
* 当仓库/分支发生更新时编译，也就是push后进行编译的意思，一般会需要选择，方便自动化构建。
* 加了GH_TOEKN等变量，value值为刚才预备工作中准备的token字符串

#### 编写.travis.yml文件

```
# 指定语言环境
language: node_js
# 指定需要sudo权限
sudo: required
# 指定node_js版本
node_js: 
  - stable
# 指定缓存模块，可选。缓存可加快编译速度。
cache:
  directories:
    - node_modules

# 指定博客源码分支，因人而异。hexo博客源码托管在独立repo则不用设置此项
branches:
  only:
    - hexo 

before_install:
  - npm install -g hexo-cli

# Start: Build Lifecycle
install:
  - npm install
  - npm install hexo-deployer-git --save

# 执行清缓存，生成网页操作
script:
  - hexo clean
  - hexo generate

# 设置git提交名，邮箱；替换真实token到_config.yml文件，最后depoy部署
after_script:
  - git config user.name "arthasmay"
  - git config user.email "arthasmay@gmail.com"
  # 替换同目录下的_config.yml文件中gh_token字符串为travis后台刚才配置的变量，注意此处sed命令用了双引号。单引号无效！
  - sed -i "s/gh_token/${GH_TOKEN}/g" ./_config.yml
  - hexo deploy
# End: Build LifeCycle
```

#### 修改下_config.yml文件的deploy节点(hexo博客目录下的，这里是卡到我了)

```
# 修改前
deploy:
  - type: git
    repo: git@github.com:arthasmay/arthasmay.github.io.git
    branch: master
```

```
# 修改后
deploy:
- type: git
  # 下方的gh_token会被.travis.yml中sed命令替换
  repo: https://gh_token@github.com/xiong-it/xiong-it.github.io.git
  branch: master
```

好了到了这里，访问[ArthasMay的技术小栈](https://arthasmay.github.io/)就可以愉快的玩耍了。以后的技术学习和研究就都记载在这里了。

### 待完善
* SEO的优化
* ~~Hexo使用图片~~

### hexo使用图片（2019.10.19）

1. 安装插件

``` bash
# hexo博客源文件根目录下运行
$ npm install https://github.com/CodeFalling/hexo-asset-image --save
```

2. 修改_congig.yml文件配置
``` yml
post_asset_folder:true
```

3. hexo n xxx 生成同名文件夹
```
![xxx](xxx/xxx.png)
```

## 参考
[Hexo遇上Travis-CI：可能是最通俗易懂的自动发布博客图文教程](https://blog.csdn.net/Xiong_IT/article/details/78675874)
可惜图挂了，后面自己补上。。。






