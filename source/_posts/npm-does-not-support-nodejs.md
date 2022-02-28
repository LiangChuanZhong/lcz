---
title: npm does not support nodejs
date: 2022-02-21 23:30:00
tags: [Node]
categories: 前端环境
---

win7不支持安装高版本node.js

win7支持最高的nodejs版本为v13.14.0

下载链接：https://nodejs.org/zh-cn/download/releases/


安装高版本的node.js后出现 npm does not support nodejs 等问题
是因为npm版本与node版本不一致
删除npm后重新安装node即可
手动删除npm：
删除默认安装路径C:\Program Files\nodejs\node_modules下的npm文件夹
