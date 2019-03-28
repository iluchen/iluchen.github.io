---
layout: post
title: toLocalString 方法思考及引申
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

    function format_number(n, l) {
      if (l === undefined) {
        l = 3 // 当m没有传参的时候给定一个默认的值为3，防止报错
      }
      var num = num.toFixed(m) // 保留三位小数
      var decimal = num - Math.floor(num) // decimal是十进制的意思。。
      var b = parseInt(n).toString();
      var len = b.length;
      if (len <= m) {
        return b;
      }
      var r = len % m;
      var RegExp1 = new RegExp("\\d{" + m + "}", "g")
      var RegExp2 = new RegExp("\\d{" + m + "}", "g")
      console.log(RegExp1, RegExp2)
      return r > 0 ? b.slice(0, r) + "," + b.slice(r, len).match(RegExp1).join(",") + '.' + decimal: b.slice(r, len).match(RegExp2)
        .join(",") + '.' + decimal;
    }
    console.log(format_number(str, 4))
