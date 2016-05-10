---
layout: post
title: "Rails 4 - Strong parameters"
date: 2013-01-06 12:55
comments: true
categories: Ruby
---

Rails 4 里引入了Strong parameters机制， 让我们看看有什么区别：

``` ruby Rails 3.2 - mass assignment
# app/models/uesr.rb
class User < ActiveRecord::Base
  attr_accessible :username, :password
end

# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    @user = User.create! params[:user]
    redirect_to @user
  end
end
```

``` ruby Rails 4 - using strong parameters
# app/models/uesr.rb
class User < ActiveRecord::Base; end

# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def create
    @user = User.create! user_params
    redirect_to @user
  end

  private
  def user_params
    params.require(:user).permit(:username, :password)
  end
end
```

以上两段代码都是为了防止用户恶意修改一些敏感字段。不同的是在 Rails 3.2 的时候，我们使用了 Mass assignment 方法， 我们在 model 中限制字段。 而 Rails 4 我们使用了 strong parameters后，对字段限制的控制权交给了 controller。比较可以看到 strong parameters 的使用会更加灵活，这样将更容易适应复杂的业务需求变化。

现在 controller 中的 params 是一个 [ActionController::Parameters](http://edgeapi.rubyonrails.org/classes/ActionController/Parameters.html) 对象。 ActiveModel 有一个 permitted 属性， 默认为 false。我们可以通过 permit！ 或者 permits 方法把该属性设置为 true。

ActionController::Parameters 有2个关键方法：

* [require(key)](http://edgeapi.rubyonrails.org/classes/ActionController/Parameters.html#method-i-require): 从 hash 中提取相应的 key，如果没有触发 ActionController::ParameterMissing 错误。
* [permit(keys)](http://edgeapi.rubyonrails.org/classes/ActionController/Parameters.html#method-i-permit): 设置被允许的属性，并把 permitted 属性设置为 true。