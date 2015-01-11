title: kaminari简要文档
date: 2015-01-11 22:22:43
tags: [rails,kaminari,分页]
---
##全局参数设置
执行以下命令，会在config/initializers下生成一个配置文件kaminari_config.rb
```ruby
rails g kaminari:config
```
这个文件控制着kaminari的全局设置，有以下参数可以定义
**default_per_page** 默认值25
**page_method_name** 分页方法的名称
**param_name** 分页参数的参数名，默认为param
**window / outer_window / left / right**这四个参数都与分页显示有关，设定了显示页码标签的方式和数量。

##分页界面定制
执行一下命令可以获取默认模版文件。目录在app/views/kaminari/下
```ruby
rails g kaminari:views default
```
各个文件描述如下
**_paginator.html.erb** 这是总的入口文件，可以通过修改它来调整显示的整体结构比如我不需要"首页"，"末页"这两个链接，那么我就在这个文件里面将对应的代码删除
**_page.html.erb** 对应页码链接
**_first_page.html.erb / _last_page.html.erb**对应"首页"和"末页"的链接
**_prev_page.html.erb / _next_page.html.erb** 对应"上一页"和"下一页"的链接
**_gap.html.erb** 空隙的显示，默认是中间的省略号(…)

>注意，不要修改文件名。

theme功能
在*app/views/kaminari*目录下创建一个文件夹，例如test，把默认的模版文件拷贝进去，修改模版文件的内容，就形成了个性化的样式。
view页面按下方写法即可调用个性样式。
```ruby
<%= paginate @blogs, :theme=>'test' %>
```
