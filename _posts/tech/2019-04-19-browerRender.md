---
layout: post
title: [转]浏览器的渲染过程及原理
categories: [Tech]
description: 转自 掘金 https://juejin.im/entry/59e1d31f51882578c3411c77
keywords: 渲染, 浏览器
---

## 浏览器渲染页面的过程

从耗时的角度，浏览器请求、加载、渲染一个页面，时间花在下面五件事情上：

1.DNS 查询

2.TCP 连接

3.HTTP 请求即响应

4.服务器响应

5.客户端渲染

本文讨论第五个部分，即浏览器对内容的渲染，这一部分（渲染树构建、布局及绘制），又可以分为下面五个步骤：

1.处理 HTML 标记并构建 DOM 树。

2.处理 CSS 标记并构建 CSSOM 树。

3.将 DOM 与 CSSOM 合并成一个渲染树。

4.根据渲染树来布局，以计算每个节点的几何信息。

5.将各个节点绘制到屏幕上。

需要明白，这五个步骤并不一定一次性顺序完成。如果 DOM 或 CSSOM 被修改，以上过程需要重复执行，这样才能计算出哪些像素需要在屏幕上进行重新渲染。实际页面中，CSS 与 JavaScript 往往会多次修改 DOM 和 CSSOM，下面就来看看它们的影响方式。

## 阻塞渲染：CSS 与 JavaScript

谈论资源的阻塞时，我们要清楚，现代浏览器总是并行加载资源。例如，当 HTML 解析器（HTML Parser）被脚本阻塞时，解析器虽然会停止构建 DOM，但仍会识别该脚本后面的资源，并进行预加载。

同时，由于下面两点：

1.默认情况下，CSS 被视为阻塞渲染的资源，这意味着浏览器将不会渲染任何已处理的内容，直至 CSSOM 构建完毕。

2.JavaScript 不仅可以读取和修改 DOM 属性，还可以读取和修改 CSSOM 属性。

存在阻塞的 CSS 资源时，浏览器会延迟 JavaScript 的执行和 DOM 构建。另外：

1.当浏览器遇到一个 script 标记时，DOM 构建将暂停，直至脚本完成执行。

2.JavaScript 可以查询和修改 DOM 与 CSSOM。

3.CSSOM 构建时，JavaScript 执行将暂停，直至 CSSOM 就绪。

所以，script 标签的位置很重要。实际使用时，可以遵循下面两个原则：

CSS 优先：引入顺序上，CSS 资源先于 JavaScript 资源。

JavaScript 应尽量少影响 DOM 的构建。

浏览器的发展日益加快（目前的 Chrome 官方稳定版是 61），具体的渲染策略会不断进化，但了解这些原理后，就能想通它进化的逻辑。下面来看看 CSS 与 JavaScript 具体会怎样阻塞资源。

## 改变阻塞模式：defer 与 async

为什么要将 script 加载的 defer 与 async 方式放到后面呢？因为这两种方式是的出现，全是由于前面讲的那些阻塞条件的存在。换句话说，defer 与 async 方式可以改变之前的那些阻塞情形。

首先，注意 async 与 defer 属性对于 inline-script 都是无效的，所以下面这个示例中三个 script 标签的代码会从上到下依次执行。

    <!-- 按照从上到下的顺序输出 1 2 3 -->
    <script async>
      console.log("1");
    </script>
    <script defer>
      console.log("2");
    </script>
    <script>
      console.log("3");
    </script>

故，下面两节讨论的内容都是针对设置了 src 属性的 script 标签。

### defer
    <script src="app1.js" defer></script>
    <script src="app2.js" defer></script>
    <script src="app3.js" defer></script>

defer 属性表示延迟执行引入的 JavaScript，即这段 JavaScript 加载时 HTML 并未停止解析，这两个过程是并行的。整个 document 解析完毕且 defer-script 也加载完成之后（这两件事情的顺序无关），会执行所有由 defer-script 加载的 JavaScript 代码，然后触发 DOMContentLoaded 事件。

defer 不会改变 script 中代码的执行顺序，示例代码会按照 1、2、3 的顺序执行。所以，defer 与相比普通 script，有两点区别：载入 JavaScript 文件时不阻塞 HTML 的解析，执行阶段被放到 HTML 标签解析完成之后。

### async
    <script src="app.js" async></script>
    <script src="ad.js" async></script>
    <script src="statistics.js" async></script>

async 属性表示异步执行引入的 JavaScript，与 defer 的区别在于，如果已经加载好，就会开始执行——无论此刻是 HTML 解析阶段还是 DOMContentLoaded 触发之后。需要注意的是，这种方式加载的 JavaScript 依然会阻塞 load 事件。换句话说，async-script 可能在 DOMContentLoaded 触发之前或之后执行，但一定在 load 触发之前执行。

从上一段也能推出，多个 async-script 的执行顺序是不确定的。值得注意的是，向 document 动态添加 script 标签时，async 属性默认是 true，下一节会继续这个话题。

## document.createElement

使用 document.createElement 创建的 script 默认是异步的，示例如下。

    console.log(document.createElement("script").async); // true

所以，通过动态添加 script 标签引入 JavaScript 文件默认是不会阻塞页面的。如果想同步执行，需要将 async 属性人为设置为 false。

如果使用 document.createElement 创建 link 标签会怎样呢？

    const style = document.createElement("link");
    style.rel = "stylesheet";
    style.href = "index.css";
    document.head.appendChild(style); // 阻塞？

其实这只能通过试验确定，已知的是，Chrome 中已经不会阻塞渲染，Firefox、IE 在以前是阻塞的，现在会怎样我没有试验。

##  document.write 与 innerHTML

通过 document.write 添加的 link 或 script 标签都相当于添加在 document 中的标签，因为它操作的是 document stream（所以对于 loaded 状态的页面使用 document.write 会自动调用 document.open，这会覆盖原有文档内容）。即正常情况下， link 会阻塞渲染，script 会同步执行。不过这是不推荐的方式，Chrome 已经会显示警告，提示未来有可能禁止这样引入。如果给这种方式引入的 script 添加 async 属性，Chrome 会检查是否同源，对于非同源的 async-script 是不允许这么引入的。

如果使用 innerHTML 引入 script 标签，其中的 JavaScript 不会执行。当然，可以通过 eval() 来手工处理，不过不推荐。如果引入 link 标签，我试验过在 Chrome 中是可以起作用的。另外，outerHTML、insertAdjacentHTML() 应该也是相同的行为，我并没有试验。这三者应该用于文本的操作，即只使用它们添加 text 或普通 HTML Element。

END