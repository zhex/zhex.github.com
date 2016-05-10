---
layout: post
title: "Android 中使用 CSS orientation 属性时的陷阱"
date: 2013-02-07 10:53
comments: true
categories: 前端开发
---

随着移动互联网时代的到来，大量的网站开始倾向于使用[响应式页面设计](http://en.wikipedia.org/wiki/Responsive_web_design)，以便于支持多种设备的访问。这一变化又给前端开发者们带来不少的问题， 大家又要开始一个个的填坑了。

相信 media query 会是大家开发响应式页面都会用到的方法。 可惜 android 对于 media query 的 orientation 属性支持的并不好。我们可以先来看看下面一段代码：

```html
<!doctype html>
<html>
  <head>
    <style type="text/css">
      @media only screen and (orientation: portrait) {
        body { background: red; }
      }

      @media only screen and (orientation: landscape) {
        body { background: green; }
      }
    </style>
  </head>
  <body>
    <input type="text" />
  </body>
</html> 
```

这段代码中定义了手机竖屏时背景色为红色， 横屏时为绿色。在 android 中测试看似一切正常。但当我们 focus 到 input 框的时候问题出现了。当我们在竖屏时背景色居然也是绿色。

我的猜测是 android 是通过屏幕的长宽比来判断是否是竖屏还是横评。 当我们 focus 到 input 框的时候，手机系统会自动弹出虚拟键盘， 此时屏幕的长宽发生变化，导致 android 会判断当前情况为横屏。同样的代码在 iphone 下测试完全通过。

所以大家在 orientation 属性的时候要格外小心， 避免这类情况对页面带来的破坏。