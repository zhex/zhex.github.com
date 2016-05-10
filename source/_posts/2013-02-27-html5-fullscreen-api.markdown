---
layout: post
title: "HTML5 Fullscreen API"
date: 2013-02-27 22:19
comments: true
categories: 前端开发
---

多年来，浏览器厂商都没有像 Flash 那样在网页端提供全屏模式调用， 原因只有一个: 安全。如果应用程序可以强制用户运行在全屏模式， 那用户将会失去对浏览器的控制，再也看不见浏览器菜单，看不到工具条， 甚至无法关闭窗口。 有些别有用心的程序员还可以通过种种手段对用户端的信息安全造成威胁。

随着HTML5标准的到来，很多浏览器开发商们终于把 Fullscreen API 带了进来。 区别于浏览器默认的 F11 按钮，Fullscreen API 并不是让整个浏览器处于全屏模式， 而是让页面中的某个元素处于全屏模式，可以是图片，视频，或者跑游戏的 Canvas 元素。而当元素被放到倒全屏模式的时候， 用户屏幕会收到在任何时候按 ESC 可以返回的提示。

Fullscreen 的调用非常简单:

```javascript
// 进去全屏模式
document.getElementById("myvideo").requestFullScreen();

// 退出全屏模式
document.cancelFullScreen();

// 当进入全屏模式的时候，document.fullScreen 将会返回 true
console.log(document.fullScreen) 
```

同时， 我们可以通过 :full-screen 尾类来对进入全屏模式的元素重新绘制样式。

对于不同的浏览器，苦逼的程序员还是不能忘记用各种前缀标签: -webkit, -moz, -ms ....., 所以我们可以写一个程序来自动执行这些方法。


```javascript
var pfx = ["webkit", "moz", "ms", "o", ""];

function runPrefixMethod(obj, method) {
	var p = 0, m, t;
	while (p < pfx.length && !obj[m]) {
		m = method;
		if (pfx[p] == "") {
			m = m.substr(0,1).toLowerCase() + m.substr(1);
		}
		m = pfx[p] + m;
		t = typeof obj[m];
		if (t != "undefined") {
			pfx = [pfx[p]];
			return (t == "function" ? obj[m]() : obj[m]);
		}
		p++;
	}
}
```

使用时的代码为：

```javascript
var e = document.getElementById("fullscreen");

e.onclick = function() {
	if (runPrefixMethod(document, "FullScreen") || runPrefixMethod(document, "IsFullScreen")) {
		runPrefixMethod(document, "CancelFullScreen");
	}
	else {
		runPrefixMethod(e, "RequestFullScreen");
	}
}
```

至于 CSS 的前缀后缀就只能依靠编辑器插件来帮助偷懒了。
