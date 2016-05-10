---
layout: post
title: "为什么我们的 Angular 应用总是很慢"
date: 2013-11-22 14:29
comments: true
categories: [前端开发, AngularJS]
---

AngularJS 作为 Google 开源的 JS 前端框架最近一直很火。 在 html 中加上几个标签， 根据格式写几句 javascript 代码， 一个单页应用就总跑起来了。其方便的上手体验， 双向数据绑定等特性给了很多前端开发者很好的亲切感。 而然随着大家逐渐用 angular 开始开发稍大一点的项目，发现做出来的应用总是非常慢，这完全颠覆了 angular 上来时给我们描绘的美好画面。既然遇到问题就要解决，让我们来看看为什么我们的 angular 应用这么慢。

## Dirty checking

为了实现页面数据的实时同步，angular 采用 dirty checking 来遍历绑定的对象，比较 value 现在和过去的值。如果 value 有发生变化，就触发 change 事件。
Digest 是执行 dirty checking 的机制， 由 $digest() 触发。 $digest() 每50ms执行一次，触发所属 scope 和其子 scope 的 dirty checking， dirty checking 又会触发 $watch(), 使整个 angular 中绑定的对象活起来。

正是因为这样频繁的触发，使得 dirty checking 有可能成为性能问题的根源。 官方给出的数据是 angular 在2000个数据绑定以上才会出现性能问题， 这个数字在大多数情况下肯定是够用了，然而在一些特殊情况，比如 grid table， 大数据量的 dropdown list，或者大数据量的列表中， 2000个数据绑定很快就会被消耗完了。

## $scope 定时器回收

当很多新手开始接触 angular 的时候并没有很好的了解其运行机制，所以经常会为了达到一些业务需求上的目的而在代码中添加定时器，并且很多人都没有对定时器作回收的习惯。在传统的 web 开发中，这常常不是问题，因为一旦页面切换，整个页面上的对象也都消失了。 而在 angular 中，所有元素都存活在单页面上，当 scope 的生命周期结束时没有办法自动清楚这些自定义的定时器，久而久之， 页面中存活着大量已经过期的定时器，内存被占用也越来越厉害。

## 开发中需要避免的事

既然已经明白了 angular 慢的原因，我们就需要在开发过程中做一些调整，避免一些导致慢的问题出现。

### 1. 不要用数据绑定来渲染列表

既然数据绑定时性能问题的根源， 我们就应该避免在特定的情况下使用，大数据列表和 grid table 就是。特别时在那些只是为了作展示，而不需要对数据做动态更新的情况下，就别用了。没有了angular， 我们还是可以用 handlebars 之类的模版引擎加上 jquery 来实现。

### 2. 不要用内联方式实行数据

ng-repeat 会在每次执行 $digest 的时候执行模版内的方法，所以为了提高效率，不要直接在模版内使用函数过滤表达式，这样会拖慢整个应用。

```html
<li ng-repeat="item in filteredItems()"> // 错误的方法，函数会被频繁调用
<li ng-repeat="item in items"> // 建议使用方法
```

### 3. 使用数据缓存

```javascript
/* Controller */
// 基础数据 
var items = [{name:"John", active:true }, {name:"Adam"}, {name:"Chris"}, {name:"Heather"}]; 

// 初始化数据
$scope.displayedItems = items;

// 过滤缓存
var filteredLists['active'] = $filter('filter)(items, {"active" : true});

// 实现过滤器
$scope.applyFilter = function(type) {
    if (filteredLists.hasOwnProperty(type){ // Check if filter is cached
        $scope.displayedItems = filteredLists[type];
    } else { 
        /* Non cached filtering */
    }
}

// Reset filter
$scope.resetFilter = function() {
    $scope.displayedItems = items;
}
```

```html
/* View */
<button ng-click="applyFilter('active')">Select active</button>
<ul><li ng-repeat="item in displayedItems">{{item.name}}<li></ul>
```

### 4. 在使用额外的模版时，用 ng-if 代替 ng-show

当通过模版或者 directive 来显示附加信息时，比如： 点击商品显示商品详细信息时， 应该使用 ng-if 代替 ng-show。 这可以减少页面内绑定对象的数量。

```html    
<li ng-repeat="item in items">
    <p> {{ item.title }} </p>
    <button ng-click="item.showDetails = !item.showDetails">Show details</buttons>
    <div ng-if="item.showDetails">
        {{item.details}}
    </div>
</li>
```

### 5. 不要使用 ng-mouseenter 和 ng-mouseleave

使用 angular 内建的 ng-mouseenter, ng-mouseleave 会导致视图闪烁。 如果想实现一些动态效果，建议使用 jQuery 的 animation。 

### 6. 使用 ng-show 来隐藏不需要的元素

在 ng-repeat 中的 filter 会为每个过滤创建一个元数据的子集。 过滤时，angular 通过调用 $destroy 方法把多余的元素从 $scope 中移除。 当过滤器发生变化的时候又把元素重新关联回 $scope. 每一次动作都会产生性能开销，多数情况下，这样做是没有问题的， 但如果操作频繁，或者数据表非常大的话，就会使性能降低。 这个时候我们可以用计算属性的方法来触发 ng-show 和 ng-hide， 这样可以明显提高性能。

```html 
<input ng-model="query"></input>
<li ng-repeat="item in items" ng-show="([item.name] | filter:query).length">{{item.name}}</li>
```

### 7. 使用 debounce 过滤数据

```javascript
/* Controller */
$scope.$watch('queryInput', function(newValue, oldValue) {
    if (newValue === oldValue) { return; }
    $debounce(applyQuery, 350);
});
var applyQuery = function() { 
    $scope.filter.query = $scope.query;
};
```

```html
/* View */
<input ng-model="queryInput"/>
<li ng-repeat= item in items | filter:filter.query>{{ item.title }} </li>
```

### 8. 销毁定时器

定时器无法被自动回收，所以我们需要自己手动回收

```javascript
var timer;

timer = $timeout(function () {
    // do something；
}, 1000);

$scope.$on('$destroy', function () {
    $timeout.cancel(timer);
});
```