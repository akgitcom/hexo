title: rails 第四天
date: 2015-01-12 09:37:39
tags: [rails,动态路由,DynamicRouter]
---
##在rails 4.1.5 里动态设置路由

需要动态获取数据路由相关数据来动态设置路由，有同学指出用路由分析库 [点我](https://github.com/rails/journey),看上去太复杂了，自己理解为模块设置路由，每次保存前重启应该就可以。
<!--more-->

> routename 为路由名称

###table:

```sql
create_table "singlepages", primary_key: "sid", force: true do |t|
    t.string   "title",          limit: 200,        null: false
    t.text     "description",                       null: false
    t.text     "content",        limit: 2147483647, null: false
    t.string   "keywords",       limit: 200,        null: false
    t.integer  "sort",                              null: false
    t.datetime "created_at",                        null: false
    t.datetime "updated_at",                        null: false
    t.string   "routename",      limit: 250,        null: false
end
```

###routes.rb

```ruby
resources :singlepages
get '/company', to: 'singlepages#show', sid: 1

```

###singelpage_controller.rb

```ruby
class SinglepagesController < ApplicationController
  def index
    @singlepages = Singlepage.order("sid DESC").page(params[:page]).per(5)
    @title = 'singlepage manage'
  end
  def show
    @singlepage = Singlepage.find_by_sid(params[:sid])
    redirect_to not_found_path unless @singlepage
  end
end
```

###model/singlepage.rb

> 每次保存前重启路由器

```ruby
class Singlepage < ActiveRecord::Base
  after_save :reload_routes
  def reload_routes
    DynamicRouter.reload
  end
end
```

###model/dynamic_router.rb

> 动态路由,包括加载和重启

```ruby
class DynamicRouter
  def self.load
    ComingSoon.application.routes.draw do
      Singlepage.all.each do |pg|
        puts "Routing #{pg.name}"
        get "/#{pg.name}", :to => "pages#show", defaults: { id: pg.id }
      end
    end
  end
  def self.reload
    ComingSoon.application.routes_reloader.reload!
  end
end
```

###config/routes.rb

> 在路由配置文件里加载动态路由模块

```ruby
Rails.application.routes.draw do
  root 'home#index'
  get 'not_found' => 'singlepages#not_found'
  DynamicRouter.load
end
```

OK。

文章来源:[Creating Dynamic Routes at runtime in Rails 4](http://codeconnoisseur.org/ramblings/creating-dynamic-routes-at-runtime-in-rails-4)