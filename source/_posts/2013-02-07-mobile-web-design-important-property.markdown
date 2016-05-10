---
layout: post
title: "移动 Web 设计： !important"
date: 2013-02-07 11:22
comments: true
categories: 前端开发
---

相信被各个浏览器折腾的够呛的前端开发者们都应该很了解 CSS 中 !important 这个属性了。 当年在 ie6 － 8 的时候为了解决各浏览器兼容性问题的时候，!important 属性就成为了一项利器。 他通过调高代码优先级，利用一些浏览器的识别度来帮助开发者达到样式兼容效果。 然而随着时间的推移，现代浏览器的兼容性越来越好， 很多创新公司的项目也只针对现代浏览器开发，需要对兼容性的控制也慢慢少了。

当今在移动互联网时代， 不少开发者们又都被老总派上阵去执行迁移原有产品页面到移动端的工作。 看着原有网页上一行行由 javascript 生成的 inline css 是不是想骂娘？ 直接修改js代码怕对原来的重量级网站带来不可预测的影响； 不修改吧页面效果又难以实现。 好吧，!important 属性又将再一次拯救你， 因为他的优先级是高于 inline css 的。看一段下面的代码：

```html
<!doctype html>
<html>
  <head>
    <style type="text/css">
      
      @media only screen and (max-device-width: 640px) {
        .block {
          color: green !important;
        }
      }
    </style>
  </head>
  <body>
    <div class="block" style="color: red">
      hello world!
    </div>
  </body>
</html> 
```

通过上面代码的演示，我们可以看到在手机上 .block 里的文字被设置成了绿色， 成功的覆盖了 inline css 设置的红色。

好了，找回我们的老朋友，继续面对移动互联网的挑战吧。