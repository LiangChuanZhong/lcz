---
title: VUE 自定义组件库搭建
date: 2022-06-07 18:07:50
tags: npm
categories: npm
---


# 前言
根据项目开发情况，子系统过多（三十多个系统），且各系统之间存在高重复性的开发内容，因此封装属于公司内部的组件库（开源npm库/个人私库是一样的，只是组件库存放的地方不一样）是非常有必要的。

<!-- more -->

企业私库搭建：[Verdaccio-搭建企业私库](https://cz-liang.github.io/lcz/2022/06/07/Verdaccio-%E6%90%AD%E5%BB%BA%E4%BC%81%E4%B8%9A%E7%A7%81%E5%BA%93/)


# 创建项目

```JS
vue create 项目名称
```

## src目录下创建packages文件夹
以分页组件（element-ui分页组件二次封装）为例：
## packages目录下新建pagination文件夹
## pagination文件夹下新建index.vue文件，用于编写组件具体内容：
```JS
<template>
  <div class="pagination-class">
    <el-pagination
      background
      :total="pageTotal.total"
      :page-sizes="[5, 10, 15, 30, 50, 100]"
      :current-page="pageTotal.pageNum"
      :page-size="pageTotal.pageSize"
      :layout="pageTotal.layout"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>
<script>
export default {
  name: 'pagination',
  props: {
    pageTotal: Object,
    requery: true
  },
  data() {
    return {}
  },
  methods: {
    // 更换总条数
    handleSizeChange(val) {
      this.$emit('handleSizeChange', val)
    },
    // 切换页数
    handleCurrentChange(val) {
      this.$emit('handleCurrentChange', val)
    }
  }
}
</script>

<style scoped>
.pagination-class {
  margin-top: 16px;
  text-align: right;
}
.el-pagination {
  padding: 0;
}
</style>

```
组件名要求驼峰命名（不用会报错），解决：
在vue.config.js文件下加配置：
```JS
module.exports = defineConfig({
  ***
  lintOnSave: false
  ***
})
```

## pagination文件夹下新建index.js文件，用于导出当前组件：
```JS
// 导入组件
import pagination from './index.vue'

// 为组件提供 install 安装方法，供按需引入
pagination.install = function (Vue) {
  Vue.component(pagination.name, pagination)
}

// 默认导出组件
export default pagination
```

##  packages目录下新建index.js文件，整合所有的组件，对外导出，即一个完整的组件库：
```JS
// 导入颜色选择器组件
import pagination from './pagination'
// import colorPicker from './color-picker'

// 存储组件列表
const components = [
  pagination,
//   colorPicker
]

// 定义 install 方法，接收 Vue 作为参数。如果使用 use 注册插件，则所有的组件都将被注册
const install = function (Vue) {
  // 判断是否可以安装
  if (install.installed) return
  // 遍历注册全局组件
  components.map(component => Vue.component(component.name, component))
}

// 判断是否是直接引入文件
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}

// export default install
export default {
  install,
  pagination,
//   colorPicker
}
```
## 修改package.json文件，scripts 中新增一条命令 npm run lib：
```JS
"scripts": {
    "serve": "vue-cli-service serve",
    ***
    "lib": "vue-cli-service build --target lib --name dop-cpn --dest lib ./src/packages/index.js"
},
```
**--target**：构建目标，默认为应用模式。这里修改为 lib 启用库模式。
**--dest**： 输出目录，默认 dist。这里我们改成 lib
**entry**: 最后一个参数为入口文件，默认为 src/App.vue。这里我们指定编译 packages/ 组件库目录。

## 执行编译库命令
```JS
npm run lib
```
## cd到lib问价下，执行npm init生成package.json
```JS
npm init
```
## 修改lib目录下的package.json文件
```JS
{
  "name": "dop-cpn",
  "version": "0.1.0",
  "description": "",
  "main": "dop-cpn.common.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```
配置 package.json 文件中发布到 npm 的字段
**name**: 包名，该名字是唯一的。可在 npm 官网搜索名字，如果存在则需换个名字。
**version**: 版本号，每次发布至 npm 需要修改版本号，不能和历史版本号相同。
**description**: 描述。
**main**: 入口文件，该字段需指向我们最终编译后的包文件。
**keyword**：关键字，以空格分离希望用户最终搜索的词。
**author**：作者
**private**：是否私有，需要修改为 false 才能发布到 npm
**license**： 开源协议

# 发布组件到开源npm

如果配置了其他镜像，先设置回npm镜像：
```JS
nrm use npm
```
## 登录npm
终端执行登录命令，输入用户名、密码、邮箱即可登录
```JS
npm login
```
## 发布到 npm
执行发布命令，发布组件到 npm
```JS
npm publish
```
* 每次发布注意包名唯一，版本号不能和历史版本号相同

# 发布组件到企业私库
结合：[Verdaccio-搭建企业私库](https://cz-liang.github.io/lcz/2022/06/07/Verdaccio-%E6%90%AD%E5%BB%BA%E4%BC%81%E4%B8%9A%E7%A7%81%E5%BA%93/)

如果配置了其他镜像，先设置回搭建的企业私库的镜像源：
```JS
nrm use dop
```
## 直接发布私库上
执行发布命令，发布组件到 npm
```JS
npm publish
```
* 每次发布注意包名唯一，版本号不能和历史版本号相同

# 使用
```JS
npm install dop-cpn
```
* 在main.js全局引入

```JS
***
import dop from 'dop-cpn'
Vue.use(dop)
***
```
这里的dop名字随便起都可以
* 在具体文件中直接使用组件
```JS
***
 <pagination :pageTotal="pageTotal"
      @handleCurrentChange="handleCurrentChange"
      @handleSizeChange="handleSizeChange"
      style="text-align:right" />
***
```

## 此版本缺点：
1. 每次执行编译npm run lib，lib被文件覆盖，lib文件下的package.json都要重新npm init生成，再手动更新包名或版本号
2. 多组件的话，无法按需引入