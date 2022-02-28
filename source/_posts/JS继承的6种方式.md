---
title: JS继承的6种方式
date: 2022-02-28 12:03:09
tags: [JS]
categories: JS
---

JS作为面向对象的弱类型语言，继承是其中非常强大的特性之一
<!-- more -->
<!-- 文章适当文职截断 -->

# JS继承的实现方式

要实现继承，首先得有一个父类，代码如下：

```JS
// 定义一个动物类
function Animal (name) {
  // 属性
  this.name = name || 'Animal'
  // 实例方法
  this.sleep = () => {
    console.log(this.name + '正在睡觉')
  }
}

// 原型方法
Animal.prototype.eat = function (food) {
  console.log(this.name + '正在吃' + food)
}
```

## 原型连继承

**核心：** 将父类的实例作为子类的原型

```JS
function Cat () {
}
Cat.prototype = new Animal()
Cat.prototype.name = 'cat'
// Test Code
var cat = new Cat()
console.log(cat.name)
```

特点：

1. 非常纯粹的继承关系
2. 父类新增的原型方法/原型属性，子类都能访问到
3. 简单，易于实现

缺点：

1. 要想为子类新增属性和方法，必须在```JS new Animal() ```这样的语句之后执行，不能放到构造器中

## 构造器

## 三

参考：[原文地址](https://www.cnblogs.com/humin/p/4556820.html)