title: rails 第一天
date: 2015-01-11 22:11:23
tags: [rails,kaminari]
---

![野三坡](http://7u2icj.com1.z0.glb.clouddn.com/github-IMG_0242.jpg)

##常用命令
```bash
rake db:schema:dump
rails g model 名称
rails g controller 名称
```

##常用gem:
```bash
gem 'mysql2'
gem 'kaminari' #分页
gem 'nokogiri' #DOM分析
```
<!--more-->
[kaminari](https://github.com/amatsuda/kaminari)
[kaminari简要文档](http://akgitcom.github.io/2014/09/11/kaminari%E7%AE%80%E8%A6%81%E6%96%87%E6%A1%A3/)

##使用方法：
controller:
```ruby
class AdminsController < ApplicationController
	def index
		@admins = Admin.order("id DESC").page(params[:page]).per(1)
	end
end
```
views:
```ruby
<% @admins.each do |b|%>
<%=b.sbname %>
<% end %>
<%= paginate @admins %>
```

数据库表名后缀添加方法：
在config/application.rb添加
```ruby
config.active_record.table_name_prefix = 'j_'
```
