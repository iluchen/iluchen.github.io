---
layout: post
title: toLocalString 方法思考及引申
categories: Tech
description: none
keywords: toLocalString
---

## 概述

重看 vue 教程的时候，发现它其中有一个方法 toLoaclString 能格式化 Date 类型的格式，感觉有点意思，就上 MDN 上查看了这个方法，感觉这个方法在有些时候还是挺有用的。

## 关于 toLocalString 

MDN 上对 toLocalString 的解释是：
> toLocaleString() 方法返回一个该对象的字符串表示。此方法被用于派生对象为了特定语言环境的目的（locale-specific purposes）而重载使用。

其中只有三个内置对象能使用这个方法：即 Number、Date、Array

他们对语法相同，都为以下这个：

> obj.toLocaleString([locales [, options]])

locales 主要是语言项， options 则为配置项（需要注意的是，带参数的可能支持性不是很好），具体查看api，以下就记录几个现实场景中可能会使用的情况；

默认当前的语言环境(浏览器语言环境)， 一般'en'、'zh' 够用了

    const date = new Date()
    date.toLocaleString('zh') // "2019/3/28 下午7:37:06"
    date.toLocaleString('en') // "3/28/2019, 7:37:06 PM"

Date 对象还有 toLocaleDateString 、 toLocaleTimeString 处理方法

    date.toLocaleDateString() // "2019/3/28"
    date.toLocaleTimeString() // "下午7:37:06"

处理数字三位用逗号隔开

    const n = 23333333
    n.toLocaleString()  // "23,333,333"

处理数字为金钱

    n.toLocaleString('zh', { style: 'currency', currency: 'CNY' }) // "￥23,333,333.00"

指定整数和小数的位数，加0补0等操作

    let m = 233.233
    m.toLocaleString('zh', { minimumIntegerDigits: 5 }) // "00,233.233"

## 自己写一个方法来实现类似的效果

以下方法针对整数的处理：

    // format_number(数值， 分割长度)
    function format_number(m, n) {
        if (n === undefined) {
          n = 3 // 当m没有传参的时候给定一个默认的值为3，防止报错
        }
        // let m = m.toFixed(n) // 保留三位小数
        let b = parseInt(m).toString()
        let len = b.length
        if (len <= n) {
          return b
        }
        let r = len % n
        let regex = new RegExp('\\d{' + n + '}', 'g')
        return r > 0
          ? b.slice(0, r) +
              ',' +
              b
                .slice(r, len)
                .match(regex)
                .join(',')
          : b
              .slice(r, len)
              .match(regex)
              .join(',')
      }
      format_number(2333333.33, 4)  // 233,3333
  
实现其他的格式就以上略作修改。

END
