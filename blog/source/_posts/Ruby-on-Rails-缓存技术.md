title: Ruby on Rails 缓存技术
date: 2015-01-29 17:04:18
tags: [rails,缓存]
---

当你网站访问量上升的时候，你可能为你的rails项目增加一些缓存应用。这个教程将告诉你关于rails缓存的方方面面，帮助你提高rails应用，而不必再为过度的cpu开销而烦心。

rails缓存有几种方式，这篇教程将分几个部分向你分别介绍如何应用不同的缓存方案，以及一些高级的有针对性的缓存应用。
<!--more-->

###首先介绍最快速的缓存应用：Page Caching，页面缓存

##1、为什么要进行缓存
（如果你已经对缓存的必要性有所了解，可以跳过本段）
ruby是一种解释性语言，这意味着ruby代码在没有被执行前，是不会编译成机器能识别的机器码的。
这个特点和php是一样的，但是java在这方面完全不同（java是先编译成机器码，后运行的）。
所以，当有人访问你的ruby程序的时候，你的代码才被读取并执行，你可以想象一下，当每秒钟有100个人访问你的代码的时候，你的程序将会消耗掉很多的系统资源。
那么改如何解决呢？
（译者：本篇文章讨论的是Rails框架中的缓存机制，Ruby是开发Rails框架的一种语言，和java，c语言一样的。）


##2、缓存
缓存是web处理(web应用)中的一个重要的设计策略，它被放置在一个临时的位置。如果有人请求了一个同样的应用，我们就可以为他提供一个应用的缓存版本。
提供一个缓存，不仅可以使我们的应用不用读取任何数据库资源，而且请求可以不必经过我们的rails应用服务。（译者：这句话很有意思，往后面看吧）
在开始缓存设计之前，有一些rails配置需要设定

##3、缓存配置
在开始缓存应用前，你需要配置你的**/config/environments/development.rb**文件 
```ruby
onfig.action_controller.perform_caching = true
```

> 默认情况下，缓存在产品模式下会被启动，上面的设置将在开发环境下启动缓存。

##4、Page Caching，页面缓存

页面缓存是最快速的一种缓存应用。那么应该在什么时候使用他呢？
1、对于所有用户都相同的页面
2、公开的页面，没有用户认证的页面
如果你的应用中有符合上述条件的，请继续阅读下面的部分。如果你的应用中没有，那么也请继续阅读，下面的内容更精彩(ps：我觉得全世界的广告都一样，不过郁闷的是deyeb目前没有一个页面符合上面的条件，除了404和500吧)

假设我们正在设计一个blog系统，页面并不经常变化，那么我们的controller可能这样来写：

ruby```
class BlogController < ApplicationController
  def list
     Post.find(:all, :order => "created_on desc", :limit => 10)
  end
  ...
```
可以看到，List这个action是查询最近的10篇blog文章，对应的web页面上将会显示查询的结果。
如果我们要对这个页面进行缓存的话，只需;

```ruby
class BlogController < ApplicationController
   caches_page :list  
   def list
     Post.find(:all, :order => "created_on desc", :limit => 10)
   end
  ...
```
"caches_page"将会在下次请求list这个action时，将页面内容存入一个html页面，放置在一个缓存目录里。

当你在mongrel里第一次运行该代码，访问这个页面的时候，你会在 */logs/development.log* log日志中发现下面的代码：

```logs
Processing BlogController#list (for 127.0.0.1 at 2007-02-23 00:58:56) [GET]
 Parameters: {"action"=>"list", "controller"=>"blog"}
SELECT * FROM posts ORDER BY created_on LIMIT 10
Rendering blog/list
Cached page: /blog/list.html (0.00000)
Completed in 0.18700 (5 reqs/sec) | Rendering: 0.10900 (58%) | DB: 0.00000 (0%) | 200 OK [http://localhost/blog/list]
```
注意这一句：*Cached page: /blog/list.htm*，
这说明，这个页面已经被加载，而且最终你看到的html页面已经被储存在 */public/blog/list.htm* 这个位置，如果你检查一下这个页面的话，你会发现上面没有一行ruby代码。

以后，如果当有访问同样的url地址的时候，加载的将是这个html页面，而不适重新加载这个controller中的action（译者：这样说更清 楚）。想像的到，加载一个静态的页面可是比加载并处理一个解释性语言要快得多，几乎快上100倍！（ps：这是真对大访问量情况下的有效的解决办法，同时 也是普通网站的一个很好的设计架构。）

但是只得注意或提醒的是，这样一个静态页面是没有执行任何ruby代码的。所以，当你的页面需要反应一些及时的用户信息，或者当前的信息，你就不能采用这种页面缓存，而是采用局部缓存。这个将在第二部分有讲解。

回到我们的model，
```ruby
caches_page :show
```
你猜，当我们访问某一篇具体的文章，比如"/blog/show/5“时，这个页面会被缓存在哪里？
答案是 这里：*/public/blog/show/5.html*
下面的例子说明页面缓存的位置

> http://localhost:3000/blog/list => /public/blog/list.html
> http://localhost:3000/blog/edit/5 => /public/edit/5.html
> http://localhost:3000/blog => /public/blog.html
> http://localhost:3000/ => /public/index.html
> http://localhost:3000/blog/list?page=2 => /public/blog/list.html
> **注意：第一行和最后一行的缓存文件是同一个，页面缓存是忽略掉url地址上的参数的。**


##5、如何缓存我的分页

为了缓存不同的页面，你需要不同的地址。（译者：我疯了，这个问题没想到，继续看吧）因为页面缓存的时候会忽略掉像/blog/list?page=2这样的参数，所以你需要使用/blog/list/2这样的地址形式，而原来我们使用的是id保存参数值，现在我们需要用page来保存参数值。
下面我们修改 */config/routes.rb*文件

```ruby
map.connect 'blog/list/:page',
    :controller => 'blog',
    :action => 'list',
    :requirements => { :page => /\d+/},
    :page => nil
```
使用了新的routes定义，我们的连接也应该改成 
```ruby
<%= link_to "Next Page", :controller => 'blog', :action => 'list', :page => 2 %>
```
最终的连接结果是"/blog/list/2"，当我们点这个连接的时候，后台会处理两件事情

1、应用将2放入page这个参数中，而不是原来id这个参数
2、缓存将生成 /public/blog/list/2.html 这个页面

> 所以，缓存分页，就要将页面参数变成页面的一部分，而不要使用地址参数的形式，他是会被忽略的。

##6、清除缓存
看完上面的内容后你可能想问，如果我添加完一篇新的文章到博客，该如何刷新/blog/list这个action呢？
来，看一下我们几分钟前创建的/blog/list.html 页面，它并不包含我们新创建的那个新文章。为了删除这个缓存，并生成一个新的缓存，我们需要让这个页面过期。为了是上面的两个页面过期，我们需要做：

```ruby
# This will remove /blog/list.html
expire_page(:controller => 'blog', :action => 'list')
```
```ruby
# This will remove /blog/show/5.html
expire_page(:controller => 'blog', :action => 'show', :id => 5)
```
显然我们需要在每一个执行了添加，删除，修改的地方粘贴上面的代码，但是还有一个更好的方法来解决问题。
（译者：头一次翻译这么长的文章，上面写的大家能看明白吗？百度空间发表的这篇文章是首发，完成后会在几个地方转载，大家多提意见）

###7、Sweepers，缓存自动清理
sweepers是一段自动删除旧的缓存的代码。为了实现这个功能，sweepers需要跟踪你的models，当你的model执行了增删改后，sweepers就会执行上面的命令。
sweepers可以在你的controllers目录创建，但是我觉得他应该隔离开。你可以在/config/environment.rb.进行个设置：

```ruby
Rails::Initializer.run do |config|
   # ...
   config.load_paths += %W( #{RAILS_ROOT}/app/sweepers )
   # ...
end
```

ps：记得重启你的服务
这样，你就可以在/app/sweepers 这个目录创建你的sweepers了。/app/sweepers/blog_sweeper.rb
的代码可以这样来写：
 
```ruby
class BlogSweeper < ActionController::Caching::Sweeper
  observe Post # This sweeper is going to keep an eye on the Post model

  # If our sweeper detects that a Post was created call this
  def after_create(post)
          expire_cache_for(post)
  end
  
  # If our sweeper detects that a Post was updated call this
  def after_update(post)
          expire_cache_for(post)
  end
  
  # If our sweeper detects that a Post was deleted call this
  def after_destroy(post)
          expire_cache_for(post)
  end
          
  private
  def expire_cache_for(record)
    # Expire the list page now that we posted a new blog entry
    expire_page(:controller => 'blog', :action => 'list')
    
    # Also expire the show page, incase we just edited a blog entry
    expire_page(:controller => 'blog', :action => 'show', :id => record.id)
  end
end
```

提示：我们可以使用“after_save”替代“after_create”和“after_update”，你自己来试一下吧！
我们还需要告诉我们的controller什么时候调用了sweepers，所以在controller中需要加入：
```ruby
class BlogController < ApplicationController
   caches_page :list, :show
   cache_sweeper :blog_sweeper, :only => [:create, :update, :destroy]
   ...
```
好了。我们可以尝试着添加一个新文章，这样在*logs/development.log:*中你会发现 

> Expired page: /blog/list.html (0.00000)
> Expired page: /blog/show/3.html (0.00000)
> 我们的sweepers已经工作了！
> **译者：这是本文的第一个重要知识**


#8、rails缓存与Apache/lighttpd的配合应用(deyeb关注的事情)
当我们将rails应用部署到产品环境的时候，我们多数采用的是apache服务器作为rails服务的前端，将rails请求转发到rails服务器上（Mongrel 或 Lighttpd）。因为我们使用了缓存静态文件的方法，所以我们需要告诉apache服务器，服务请求的是否存在静态页面缓存，如果有，则直接读取静态文件，而不用请求rails服务器了。（deyeb的设计如果这么简单就好啦）
httpd.conf的配置
```apache
<VirtualHost *:80>
  ...
  # Configure mongrel_cluster
  <Proxy balancer://blog_cluster>
    BalancerMember http://127.0.0.1:8030
  </Proxy>

  RewriteEngine On
  # Rewrite index to check for static
  RewriteRule ^/$ /index.html [QSA]

  # Rewrite to check for Rails cached page
  RewriteRule ^([^.]+)$ $1.html [QSA]

  # Redirect all non-static requests to cluster
  RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
  RewriteRule ^/(.*)$ balancer://blog_cluster%{REQUEST_URI} [P,QSA,L]
  ...
</VirtualHost>
```
lighttpd 的配置写法
```lighttpd
server.modules = ( "mod_rewrite", ... )
url.rewrite += ( "^/$" => "/index.html" )
url.rewrite += ( "^([^.]+)$" => "$1.html" )
```
代理服务器将会查看你放置在 /public 目录下的静态文件，但是你可能想将这个缓存位置改变一下，以保持独立性。那就继续看下面的介绍。


##9、更改你的缓存放置位置
首先需要在你的 */config/environment.rb* 中添加下面的代码

config.action_controller.page_cache_directory = RAILS_ROOT + "/public/cache/"
这是告诉rails服务你需要将缓存文件放置在/public/cache这个目录下，你还需要重写一下httpd.conf规则。
```apache

  # Rewrite index to check for static
  RewriteRule ^/$ cache/index.html [QSA]

  # Rewrite to check for Rails cached page
  RewriteRule ^([^.]+)$ cache/$1.html [QSA]
```
##10、清除你的局部或整体缓存

(原文写的太啰嗦)当你采用了缓存技术时，有时候可能在增删改一个model的时候，需要过期所有的页面缓存，比如当你的所有页面都包含一个最新文章列表的时候。
一个方法是删除你所有的缓存文件，首先可以像上面介绍的那样，移动你的缓存目录，然后按照下面的方法写一个sweepers
```ruby
class BlogSweeper < ActionController::Caching::Sweeper
  observe Post

  def after_save(record)
    self.class::sweep
  end
  
  def after_destroy(record)
    self.class::sweep
  end
  
  def self.sweep
    cache_dir = ActionController::Base.page_cache_directory
    unless cache_dir == RAILS_ROOT+"/public"
      FileUtils.rm_r(Dir.glob(cache_dir+"/*")) rescue Errno::ENOENT
      RAILS_DEFAULT_LOGGER.info("Cache directory '#{cache_dir}' fully sweeped.")
    end
  end
end
```
FileUtils.rm_r 将会删除你所有的缓存文件，当然你也可以逐步删除你的缓存，比如当你想删除/public/blog下面的缓存时，可以：

```ruby
cache_dir = ActionController::Base.page_cache_directory
FileUtils.rm_r(Dir.glob(cache_dir+"/blog/*")) rescue Errno::ENOENT
```
如果你觉得File Utilities对你实在难于把握，你可以尝试一下 Charlie Bowman 的 broomstick plugin
只需一个简单的调用，就可以删除你每一个controller或action缓存。
（译者：有必要也会翻译一下上面的这篇文章《Modules, Mixins and Inheritance》，直译 为 组件，混入，和继承，但是这样的计算机编程属于还是用英文的好）

##11、更高级的缓存？

页面缓存对于像deyeb这样的大型网站，是比较复杂的。（译者：要了朕的亲命了，不然也不会翻译写外文寻找光明了！）下面还有一些其他的方法
Rick Olson (aka Technoweenie)写了个 Referenced Page Caching Plugin ，可以使用数据库表进行页面缓存。
Max Dunn 写了篇非常好的文章，Advanced Page Caching ，教你如何根据用户的角色，使用cookies进行缓存应用的。
最后，貌似没有发现页面缓存xml文件的好方法，Mike Zornek的wrote about his problems 指出了一个处理方法。Manoel Lemos way to do it using action caching
我们将在下一章介绍action缓存。

##12、如何测试缓存

没有现成的方法，Luckily Damien Merenne 有一个 swank plugin ，看一下吧。

##13、本章结束

页面缓存应该在你的应用中广泛的使用，毕竟它可以提供很快的相应速度。但是当你的项目始终需要进行用户身份验证和身份显示的时候，这样的处理就有局限了！
