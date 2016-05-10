---
layout: post
title: "加速编译速度，使用 Gulp 代替 Grunt"
date: 2014-05-08 15:33
comments: true
categories: 前端开发
---


用了差不多一年的 [grunt](http://www.gruntjs.com)，体会到 grunt 的强大，的确帮助我们团队在前端的开发过程节约了很多时间，不过对 grunt 的执行速度我还是有点不满意。 

我是模块化开发的推崇者， 所以在开发过程中我喜欢把不同的模块写入多个文件， 在 html 页面中只是引入合并/编译后的文件。在后台用 connect ＋ watch 开启服务器监听，实时合并编译，配合 livereload，这样就能免去在增加一个新模块文件的时候，我还要记得去 html 中引入该文件。而此时 grunt 的执行速度却满足不了我的要求，经常我 `CMD＋S` 以后要等待3，4秒文件才编译完成（我用的是SSD哦）。本来为了方便开发，而现在我的时间都在等待中度过了。

最近我发现了 [gulp](http://www.gulpjs.com)， 他借助 nodejs 中优秀的 stream 特性，把多个 task 之间用数据流联系了起来。以往我们在 grunt 中多个 task 顺序执行常常需要借助一个 tmp 文件作为过度，而在 gulp 中直接读取上一个 task 返回的流就可以了，这样减少了一次读文件的时间，大大提高了执行效率。


```javascript
// sample gulpfile
var path = require('path'),
    gulp = require('gulp'),
    plugins = require('gulp-load-plugins')();

//.......

gulp.task('script', function () {
    return gulp.src(paths.scripts)
        .pipe(plugins.concat('all.js'))
        .pipe(gulp.dest('app/js'))
        .pipe(plugins.connect.reload());
});

//......

gulp.task('default', ['script']);

```

把工具替换成 gulp 以后， 同样的合并任务现在只要600ms就可以完成。一下子感觉到懒人的世界那个快乐啊。

另外， gulpfile 的编写更像是在写 js 代码，而不是 grunt 的 json 配置方式，同样的任务 gulpfile 明显要简约很多。相对 gulp， grunt 现在最大的优势就是庞大的社区和丰富的插件。听说 grunt 也将在 0.5 版本以后用流的方式来实现 task 之间的关联。如果 gulp 的现有插件能够满足你的需求，强烈建议在 grunt 发布新版本前试试 gulp。