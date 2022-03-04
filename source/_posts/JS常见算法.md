---
title: JS常见算法
date: 2022-03-04 23:52:59
tags: [JS, 算法]
categories: JS
---


常见算法积累

<!-- more -->

### 找出 [1, 2, 1, 3, 2, 3, 4, 2, 5] 数组中出现最多的数字，出现几次?

```JS
let arr = [1, 2, 1, 3, 2, 3, 4, 2, 5]
function moreNum(arr) {
    let obj = {}
    let num = 0 // 出现次数
    let akey = null  // 出现最多的数字
    arr.forEach(im => {
        obj[im]?
        obj[im] = obj[im] + 1
        :
        obj[im] = 1
    })
    console.log(obj)
    for(let key in obj) {
        if(obj[key] > num){
            num = obj[key]
            akey = key
        }
    }
    console.log(akey,num)
}
moreNum(arr)
```
解题思路：
- 遍历数组把每个数字作为对象的key,存在key,则该值+1，不存在则为1

***

### 判断 str="([]{})"字符串中符号是否正确匹配?
指的是符号一一对应，()、{}、[]
```JS
let str = '([]{})'
function isBalance(str) {
    let arr = str.split('')
    let new_arr = []
    for(let i = 0;i<arr.length;i++) {
        // 把左括号加入栈（数组）中
        if(arr[i] == '(' || arr[i] == '{' || arr[i] == '['){
            new_arr.push(arr[i])
        }else {
            // 然后判断出现的右括号是否和最后加入栈中（数组最后一项）的左括号匹配
            if(arr[i] == ')' && new_arr.pop() != '(') return false
            if(arr[i] == '}' && new_arr.pop() != '{') return false
            if(arr[i] == ']' && new_arr.pop() != '[') return false
        }
    }
    return new_arr.length === 0
}
console.log(isBalance(str))
```
解题思路：
- 首先字符串先分割成数组，利用 __“栈”的先进后出__ 原理判断符号是否成对出现
- 关键：数组 __.pop()__ （删除数组最后一项并返回该项）方法返回数组最后一项，如['(','{'].pop()返回'{'