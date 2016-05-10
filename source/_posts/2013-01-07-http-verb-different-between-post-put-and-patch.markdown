---
layout: post
title: "HTTP Verb: POST, PUT和PATCH的区别"
date: 2013-01-07 22:16
comments: true
categories: Ruby
---

一直一来对HTTP Verb的认识是 GET, POST, PUT, DELETE。 随着Rails 4的到，又让我认识了 PATCH。接踵而来的问题当然是 POST, PUT 和 PATCH 的区别了。

先来看看官方定义：

- **POST** to create a new resource when the client cannot predict the identity on the origin server (think a new order)
- **PUT** to override the definition of a specified resource with what is passed in from the client
- **PATCH** to override a portion of a specified resource in a predictable and effectively transactional way (if the entire patch cannot be performed, the server should not do any part of it)

显而易见， post表示新增，  put可以认为是完整替换， 而patch的作用是部分替换。

```ruby
# POST /items
def create
  @item = Item.new
  @item.attributes = { :name => params[:name],
                      :image => params[:image] }
  @item.save
end
```

这里我们利用 POST 创建了一个服务器端从来就没有的 item。


```ruby
# PUT /items/{id}
def replace
  @item = Item.find_by_id(params[:id])

  unless @item  # if @item.nil?
      @item = Item.new
      @item.id = params[:id]
  end

  @item.attributes = { :name => params[:name],
                       :image => params[:image] }
  @item.save
end
```

如果在上面的代码中，我们没有传递 :image, 那更新后 :image 将会是 nil

```ruby
# PATCH /items/{id}
def patch
  @item = Item.find(params[:id])
  @item.attributes = params.slice(:name, :image)
  @item.save
end
```

上面的代码中，我们只更新:name, :image, 其他部分内容不会被更新，也不会被滞空。