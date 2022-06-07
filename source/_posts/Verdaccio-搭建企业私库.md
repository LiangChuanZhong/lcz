---
title: Verdaccio 搭建企业私库
date: 2022-06-07 15:13:18
tags: npm
categories: npm
---



## 前言
私有 npm 库，我想是每个团队都会实践和经历的一个阶段。实现私有 npm 的方式有很多种，例如基于私有 Git 仓库、基于 npm 官方提供的私有功能（付费）、Verdaccio 等等。但是，综合比较各种因素下来（不要钱、还好用），Verdaccio 都略胜前面两者。

<!-- more -->
<!-- 文章适当文字截断 -->

# 一、安装

Verdaccio 的安装启动过程较为简单。首先是全局安装 Verdaccio：
```JS
npm i -g verdaccio
```
默认安装路径：C:\Users\EDY\AppData\Roaming\verdaccio
* 启动
```JS
verdaccio 
```
此时浏览器打开可访问： http://localhost:4873/
# 二、配置修改
config.yaml默认配置
```JS
storage: ./storage
plugins: ./plugins
web:
  title: Verdaccio
auth:
  htpasswd:
    file: ./htpasswd
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
packages:
  '@*/*':
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
  '**':
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
server:
  keepAliveTimeout: 60
middlewares:
  audit:
    enabled: true
logs:
  - { type: stdout, format: pretty, level: http }
```
这里我们来逐个认识一下默认配置中的几个值的含义：
**storage**   已发布的包的存储位置，默认存储在 ~/.config/Verdaccio/ 文件夹下
**plugins**   插件所在的目录
**web**   界面相关的配置
**auth**  用户相关，例如注册、鉴权插件（默认使用的是 htpasswd）
**uplinks**   用于提供对外部包的访问，例如访问 npm、cnpm 对应的源
**packages**  用于配置发布包、删除包、查看包的权限
**server**  私有库服务端相关的配置
**middlewares**   中间件相关配置，默认会引入 auit 中间件，来支持 npm audit 命令
**logs**  终端输出的信息的配置

这里我们要做一些处理，前面监听的地址为http://localhost:4783/,但是公网上访问需要设置为0.0.0.0:4873
* 公网访问需要在配置上加：
```JS
listen: 0.0.0.0:4873
```

在生产环境下，私有 npm 库还需要具备以下 2 个功能：
* 支持对 npm 包的搜索
* 严格的权限把控，npm 包的访问只能是已注册的用户。并且在一些场景下，需要删除用户
1. 首先前往verdaccio配置文件config.yaml中加入
```JS
search: true
```
2. 权限把控 
* access 控制包的访问权限
* publish 控制包的发布权限
* unpublish 控制包的删除权限
相关的权限，它有三个可选值 $all（所有人）、$anonymous（未注册用户）、$authenticated（注册用户）。
作为企业私库，需要修改下权限，为如下：
```JS
packages:
  '@*/*':
    access: $authenticated
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs

  '**':
    access: $authenticated
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
```

以上修改后的完整配置为：
```JS
storage: ./storage
plugins: ./plugins
web:
  title: Verdaccio
auth:
  htpasswd:
    file: ./htpasswd
    max_users: 1000 #默认用户为1000，改为-1，禁止注册
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
packages:
  '@*/*':
    access: $authenticated
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs

  '**':
    access: $authenticated
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
server:
  keepAliveTimeout: 60
middlewares:
  audit:
    enabled: true
logs: { type: stdout, format: pretty, level: http }

listen: 0.0.0.0:4873

search: true
```



修改完了配置，到verdaccio目录下通过$ verdaccio -c config.yaml更新一下
重启Verdaccio就可以通过ip访问：http://192.168.18.170:4873/
后续发布包也可以搜索

# 三、基本使用
#### 注册用户
```JS
npm adduser --registry http://192.168.18.170:4873/
```
接着，它会要求你填写用户名、密码和邮箱，用于登陆私有 npm 库
注册的用户信息会存储在 ~/.config/verdaccio/htpasswd 文件中，
想要删除用户可直接在htpasswd文件中删除
#### 添加、切换源
nrm 来切换源
```JS
npm i -g nrm
```
添加源
```JS
nrm add dop http://192.168.18.170:4873/
```
dop 为自定义源名称
查看源列表：
```JS
nrm ls
```
使用内部自定义源：
```JS
nrm use dop
```

#### 使用自定义源发布npm包
在组件库代码项目中执行
```JS
npm publish
```
组件库封装并发布看另外一篇文章

##注意
* **以上为本地搭建，真正发布到服务器在生产环境下使用 是直接复制C:\Users\EDY\AppData\Roaming\verdaccio文件夹？还是重装？后续实验一下**
同域下测试过，用其他电脑通过ip访问可下载依赖包并正常使用






### 参考：
[Verdaccio，cnpm，git仓库搭建企业级私库](https://juejin.cn/post/6844904033354776590)
[记录windows环境下用verdaccio搭建npm私有库](https://blog.csdn.net/weixin_43249693/article/details/84453017?spm=1001.2101.3001.6650.19&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-19-84453017-blog-114261826.pc_relevant_antiscanv2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-19-84453017-blog-114261826.pc_relevant_antiscanv2&utm_relevant_index=25)
[使用 Verdaccio 搭建一个企业级私有 npm 库](https://blog.csdn.net/qq_42049445/article/details/114261826)
