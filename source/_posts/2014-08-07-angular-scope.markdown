---
layout: post
title: "理解 Angular Scope"
date: 2014-08-07 09:55
comments: true
categories: 前端开发
---


很多人开始用 `angular` 的时候都觉得 `$scope` 这个玩意很神奇，但又很难理解其工作原理，所以我希望通过这篇文章来说说 angular scope，让大家可以更好的写出 angular 代码。

## $rootScope

根据词义就能了解到 $rootScope 是所有 $scope 的顶级节点，所有的 $scope 都是继承自他。 $rootScope 会被创建在我们设置 `ng-app` 属性的 DOM 元素上。

```html
<html ng-app> <!-- 在这里创建了 $rootScope -->
<body>
</body>
</html>
```

我们可以很方便的开打 console，测试打印节点的 scope： 

```javascript
// 在 console 打印 $rootScope
angular.element(document.documentElement).scope()
```

如果我们来一段这样的代码：

```html
<!doctype html>
<html ng-app>
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.0-beta.13/angular.min.js"></script>
  </head>
  <body>
    <div>
      <label>Name:</label>
      <input type="text" ng-model="yourName" placeholder="Enter a name here">
      <hr>
      <h1>Hello {{yourName}}!</h1>
    </div>
  </body>
</html>
```

这里有一个 叫 ng-model 的 directive ， directive 的工作就是把DOM 元素和 scope连接起来，让开发者在这里对 DOM 元素的行为进行操作。 比如这里的 ng-model 就把 input 元素绑定了 $rootScope 上，并加入了一个 `yourName` 的属性.

angular 还会为元素加上 change 事件，当 change 事件被触发的时候，angular 会用 input 元素的值去更新 scope 中 `yourName` 的值，并且通知其他相关元素作更新。 这一切都是由 angular 的 dirty checking 机制完成的。在这里原本 $rootScope 是没有 ｀yourName` 对象的， angular 会自动为你创建。 

这就是所谓的双向数据绑定（2-way data binding），当改变 model 的时候， 页面上的相关元素都会响应的变化。


## Controller 中的 $scope 

要知道 $scope 在 controller 中如何工作的，肯定先要知道 controller 是干什么的。 在 angular 中 controller 就是把 数据 model 传送到 view 层提供展示。非常简单，直接。 当 view 问 controller 要数据的时候，controller 就会根据 view 的要求，乖乖的把数据上交。

通常我们都希望让 controller 保持短小精悍，便于测试。

```javascript
angular.module("MyModule")
    .controller("FooCtrl", function($scope) {
        $scope.user = {
            name: "Foo"
        };

        $scope.doFoo = function() {
            // Do something!
        };
    });
```

当我们用 `ng-controller` 绑定页面元素的时候，我们实际创建了一个继承自 $rootScope 的 子 scope。

## 嵌套式 Controller

我们知道 angular 中的 controller 是可以嵌套的，controller 创建的 scope 有可能继承自其他的 scope， 所以我们在使用的时候也要非常小心。当我们在父级 scope 上创建原始值（string， number， boolean）的时候，子级 scope 并不能很好的维护他。 当子级 scope 去修改父级 scope 的原始值的时候， angular 会打破继承， 在子级 scope 复制一个值。 所以我们最好使用对象， 比如`ng-model="user.name"`, 这样就不会打破继承关系。

另外我们可以使用 `controllerAs` 来解决继承问题，比如如下代码：

```javascript
angular.module("MyModule")
    .controller("FirstCtrl", function() {
        this.user = {
            name: "Dr. Evil"
        };
    })
    .controller("SecondCtrl", function() {
        this.lair = {
            name: "Underground"
        };
    });
```

```html
<div ng-controller="FirstController as first">
    <div ng-controller="SecondController as second">
        {{first.user.name}}
        {{second.lair.name}}
    </div>
</div>
```

angular 在 1.2 版本后可以通过 controllerAs 提供一个 controller 的昵称很好的解决继承问题，开发者在使用的时候可以明确指定要使用哪个 scope 中的对象。

其实刚才的代码就是把 $scope.first 挂上了 controller 的 this，并没有多么神奇，上面的代码等同于：

```javascript
angular.module("MyModule")
    .controller("FirstCtrl", function($scope) {
        $scope.first = this;

        $scope.first.user = {
            name: "Dr. Evil"
        };
    })
    .controller("SecondCtrl", function($scope) {
        $scope.second = this;

        $scope.second.lair = {
            name: "Underground"
        };
    });
```

使用 controllerAs 是不是会觉得逻辑更清晰一点？

## directive 中的 $scope

创建自定义 directive 是一个经常容易出问题的地方，这却恰恰又是 angular 的强大所在。首先我们在创建 directive 的时候可以通过 `scope` 属性来设置该 directive scope 的控制范围。

```javascript
angular.module("MyModule")
  .directive("MyDirective", function() {
    return {
      scope: "false|true|{}"
    }
  });
```

默认情况下 `scope: flase`。

当我们想创建可复用的 directive 的时候， scope 的默认设置可能是一个危险的信号。因为此时 directive 其实是使用父级的 scope， 当我们想在 directive 创建或操作属性时候，其实是在 父级 scope 进行，这样在复用性上会有潜在的问题。

```javascript
angular.module("MyModule")
  .directive("MyDirective", function() {
    return {
      scope: false,
      link: function(scope, el) {
        scope.myNewProp = "其实我是上面scope的";
      }
    }
  });
```

当然，我们也可以把 scope 设置成 true。 这样 directive 的行为就和 ng-controller 一样了， 他会创建一个全新的 子级 scope 继承自父级。 这样你就可以访问父级 scope 的属性，也可以在自己的 scope 中创建新属性。

```javascript
angular.module("MyModule")
  .directive("MyDirective", function() {
    return {
      scope: true,
      link: function(scope, el) {
        scope.someParentObject.foo = "我还是可以改变父级元素的属性";

        scope.someLocalObject = {
          foo: "我是本地scope的属性"
        }
      }
    }
  });
```

我们可以从上面代码看到，当我们访问父级属性的时候，我们还是可以对父级属性修改，当创建新属性的时候，我们其实是在自己新的 scope 中创建。


相对于来说， isolate scope 就要附在一些。

```javascript
angular.module("MyModule")
  .directive("MyDirective", function() {
    return {
      scope: {
        property: "=",
        expression: "@"
        action: "&"
      },
      link: function(scope, el) {
      }
    }
  });
```

我们可以把上面代码中 scope 设置的三种符号认为是三种过滤器，下面让我们看看这三种过滤器各自的效果。

### scope: =

```javascript
angular.module("MyApp", [])
  .controller("UserCtrl", function($scope) {
    $scope.loggedInUser = {
      name: "Austin Powers"
    }
  })
  .directive("myUserDirective", function() {
    return {
      restrict: "E",
      template: "<input ng-model='user.name' /></div>",
      scope: {
        user: "="
      },
      link: function(scope) {
        console.log(scope.user) // { name: "Austin Powers" }
      }
    };
  });
```
有了上面的代码， 我们就可以这样`<my-user-directive user="loggedInUser"></my-user-directive>`使用 directive 了。 通过“＝”我们可以把父级的`loggedInUser`属性作为`user`传到子级，并且在 direcitve 和父级 scope 间建立双向绑定。使用“＝”的时候，传值永远是字符串值，就像这里的 loggedInUser 是父级 scope 的属性。如果我们改变`ng-model="user.name"`的值，父级 UserCtrl 中的 loggedInUser 也会随之改变。

### scope: @

相对于 “＝” 的双向绑定， “@” 实现的是单向绑定，对 direcitve scope 中属性的修改，不会影响到父级 scope 的属性。属性的传递是字符串，或者是在父级生成的字符串。

```javascript
angular.module("MyApp", [])
  .controller("UserCtrl", function($scope) {
    $scope.loggedInUser = {
      firstName: "zhe",
      lastName: "xu"
    };
  })
  .directive("myUserDirective", function() {
    return {
      restrict: "E",
      template: "{{fullName}}",
      scope: {
        fullName: "@name"
      }
    };
  });
```

```html
<my-user-directive name="{{loggedInUser.firstName}} {{loggedInUser.lastName}}">
</my-user-directive>
```

上例中我们在父级 scope 拼装了一个 name 属性赋给 directive 的 fullName, fullName 会随着父级 scope 中的 firstName 和 lastName 改变而改变。

### scope: &

“&” 过滤器最牛的地方是可以直接调用父级 scope 的表达式

```javascript
angular.module("MyApp", [])
  .controller("MathCtrl", function($scope) {
    $scope.add = function(x, y) {
      return x + y;
    };
  })
  .directive("myAddThings", function() {
    return {
      restrict: "E",
      template: "{{result}}",
      scope: {
        localFn: "&fn"
      },
      link: function(scope) {
        scope.result = scope.localFn({
          x: 1,
          y: 2
        });
      }
    };
  });
```

```html
<my-add-things fn="add(x, y)"></my-add-things>
```

上面的例子中， 我们把父级的 add 方法作为表达式写入 html，directive 通过 “&” 从父级 fn 继承表达式并赋给 localFn，然后我们就可以在dirctive 中使用 add 方法了， 而且可以直接传 dirctive scope 中的值进去。这个过滤器可以帮助我们更好的使用父级方法，如果几个不同的 directive 中需要用到公用方法，就可以用 “&” 来避免重复定义。


## 结束语

学 angluar 的路漫漫兮，scope只是其中的一块小蛋糕，希望在整个学习过程中，这篇文章可以对你有帮助。
