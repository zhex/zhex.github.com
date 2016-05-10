---
layout: post
title: "通过 monkey path 让 gridster 在 angular 中工作"
date: 2013-12-04 21:27
comments: true
categories: [前端开发, AngularJS]
---

[Gridster](http://www.gridster.net) 是一个非常酷的可拖拽的栅格插件。最近我也把他用到了我的项目中。然而 Gridster 的实现和 angular v1.2.3 格格不入，在删除栅格的时候总是会引发莫名其妙的错误。 

经过研究后发现， Gridster 在初始化栅格的时候会重新 append 元素，而这个行为却会导致 ng-repeat 内的元素关系错乱。尝试了各种方法也无法使其工作，放弃这么好的插件又不甘心，好吧，开始动修改插件的念头吧。 直接修改插件肯定是不可取的方法， 因为我们项目中的插件都是用 [bower](http://bower.io/) 管理，直接修改插件会导致以后升级困难，不便于项目的维护，这可是丢了西瓜，拣了芝麻。所以这里我借鉴了 ruby 中的 monkey patch 方法来打补丁，这样不但可以保持原来的 gridster 插件完整，又可以方便的实现自己想要的改进。

ok，现在让我们开工开始打 patch。

发现主要问题出在 add_widget 这个方法上：

```javascript jquery.gridster.js
fn.add_widget = function(html, size_x, size_y, col, row, max_size) {
  // .......

  var $w = $(html).attr({
    'data-col': pos.col,
    'data-row': pos.row,
    'data-sizex' : size_x,
    'data-sizey' : size_y
  }).addClass('gs-w').appendTo(this.$el).hide();

  // ....
};
```

现在我们创建 gridster.monkey_path.js 文件

```javascript gridster.monkey_path.js
(function ($) {

// 重命名原方法，以备其他用途
$.Gridster.add_new_widget = $.Gridster.add_widget;

// 创建应用覆盖
$.Gridster.add_widget = function(html, size_x, size_y, col, row, max_size) {
  // .......

  var $w = $(html).attr({
    'data-col': pos.col,
    'data-row': pos.row,
    'data-sizex' : size_x,
    'data-sizey' : size_y
  }).addClass('gs-w').hide(); // 除去 appendTo(this.$el)

  // ....
};

})(jQuery);
```
最后我们按照顺序加载 js 文件， 一切就搞定了， gridster 现在可以在 angular 中工作了。
```html
<script src="bower_components/gridster/dist/jquery.gridster.js"></script>
<script src="scripts/utils/gridster.monkey_patch.js"></script>
```