---
layout: post
title: "Angularjs 中的 provider, factory 和 service"
date: 2013-08-03 16:27
comments: true
categories: [前端开发, AngularJS]
---


相信开发过 angularjs 的兄弟们都会被 provider， factory 和 service 困扰，有没有？本人开始也是一头雾水，看似都是差不多的东西，各有什么用呢？经过去外国社区大量搜索以后终于找到了答案：

其实 provider， factory， service 都属于 provider， 对的，你没有听错。难道 google 的那帮大牛疯了，没事搞3个一样东西出来？还是让我们先来看看代码分析吧。

## 什么是 provider ?

provider 可以为应用提供通用的服务，形式可以是常量，也可以是对象。

比如我们在 controller 里注入进来的 $http, $scope 都可以认为是 provider。 

```javascript
app.controller('MainCtrl', function ($scope, $http) {
  $http.get(....).then(.....);
}
```

现在让我们自己来定制一个 provider

```javascript
// 方法 1
$provide.provider('test', {
  n: 3;
  $get: function () { 
    return n;
  };
});

// 方法 2
$provide.provider('test', function () {
  this.n = 3;
  this.$get = function () { 
    return n;
  };
});

// 使用
app.controller('MainCtrl', function ($scope, test) {
  $scope.test = test;
});
```

让我们看看 provider 的内部实现代码

```javascript
function provider(name, provider_) {
  if (isFunction(provider_)) {
    provider_ = providerInjector.instantiate(provider_);
  }
  if (!provider_.$get) {
    throw Error('Provider ' + name + ' must define $get factory method.');
  }
  return providerCache[name + providerSuffix] = provider_;
}
```

可以看到 provider 的基本原则就是通过实现 $get 方法来进行单例注入，使用时获得的就是 $get 执行后的结果。

## factory

那如果每次都要写一个 $get 是不是很麻烦？ OK，所以我们有了 factory。 factory 可以说是 provider 的变种， 方法中的第二个参数就是 $get 中的内容。

```javascript
// 定义 factory 
$provide.factory('dd', function () { 
  return new Date();
});

// 使用
app.controller('MainCtrl', function ($scope, dd) {
  $scope.mydate = dd;
});
```

factory 的实现源代码：

```javascript
function factory(name, factoryFn) { 
  return provider(name, { 
    $get: factoryFn 
  }); 
}
```

## service 

在 factory 的例子中我们还是需要 new 一个对象返回，而 service 就更简单了，这一步都帮你省了， 他的第二个参数就是你要返回的对象类， 也就是 new 的哦给你工作都不用你做了。够清爽吧？

```javascript
// 定义 service 
$provide.service('dd', Date);
```

下面是 service 的实现源代码：

```javascript
function service(name, constructor) {
  return factory(name, ['$injector', function($injector) {
    return $injector.instantiate(constructor);
  }]);
}
```

然后 factory 和 service 带来代码精简的同时也损失了一些特性。 provider 定义的服务是可以通过模块 config 来配置的。

