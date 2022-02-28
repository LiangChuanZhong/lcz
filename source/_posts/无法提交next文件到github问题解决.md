---
title: 无法提交next文件到github问题解决
date: 2022-01-17 15:17:12
tags: [Hexo]
categories: 其他
---

无法提交根本原因是next主题也是一个repo。

### 解决方法

1. 剪切 themes/next/.git文件夹到其它处

2. 从暂存区删除该文件夹
git rm --cache themes/next

3. 使用git status查看状态

4. 三步走: -->git add .  -->git commit -m "" -->git push

5. 再移回themes/next/.git文件夹