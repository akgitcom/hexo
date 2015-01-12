title: Rails 4.1.6 使用Devise 和Simple Captcha 验证码
date: 2015-01-12 09:45:52
tags: [rails,devise,simple captcha]
---

##rails simple_captcha 验证码实现

###用到的gem有

```rails
gem "galetahub-simple_captcha", :require => "simple_captcha"
gem "mini_magick"
```
<!--more-->
执行**bundle**

重写**devise**的**controller**方法我把**devise**的方法放到*/admin*里了

```rails
devise_for :users,
           :path => "/admin/users",
           :controllers => { :sessions => "devise_hack/sessions",:registrations => "devise_hack/registrations" },
           :path_names  => { :sign_in  => 'login', :sign_out => 'logout', :sign_up => 'sign_up'}
```

###执行下面命令安装simple_captcha

```rails
rails generate simple_captcha
rake db:migrate
```

生成**_simple_captcha.html.erb**文件，该文件为验证码的局部视图模板。修改局部视图**/vews/devise/sessions/new.html.erb**，添加一个切换验证码链接。

```rails
<%= show_simple_captcha %>
```

##注册和登陆都需要验证码，需要重构两个controller的create方法

在**controllers/devise_hack/registrations_controller.rb**中

```rails
class DeviseHack::RegistrationsController < Devise::RegistrationsController
    include SimpleCaptcha::ControllerHelpers
    def create
    if simple_captcha_valid?
      super
    else
      build_resource
      #clean_up_passwords(resource)
      params[:user][:password] = nil
      flash.now[:alert] = "验证码错误!"
      respond_with_navigational(resource) { render :new }
    end
  end
end
```

> skip_before_filter :require_no_authentication, :only => [:new]
只有在登陆的时候调用此方法,如果没有会一直重定向到注册页面

在**controllers/devise_hack/sessions_controller.rb**中

```rails
class DeviseHack::SessionsController < Devise::SessionsController
  include SimpleCaptcha::ControllerHelpers
  skip_before_filter :require_no_authentication, :only => [:new]
  def create
    if simple_captcha_valid?
      super
    else
      build_resource
      flash[:error] = "Captcha has wrong, try a again."
      redirect_to '/admin/users/login'
      #respond_with_navigational(resource) { render :new }
    end
  end
  def build_resource(hash=nil)
    self.resource = resource_class.new_with_session(hash || {}, session)
  end
end
```

##遇到的问题:

ImageMagick: Error while running convert: convert: unable to read font

> 安装 $ brew install ghostscript

验证码不验证问题（具体记不清了。。）

> 将render换成redirect_to

##参考文章：

[http://www.cnblogs.com/itmangelihai/p/3254608.html](http://www.cnblogs.com/itmangelihai/p/3254608.html)
[https://github.com/plataformatec/devise/wiki/How-To%3a-Use-Recaptcha-with-Devise](https://github.com/plataformatec/devise/wiki/How-To%3a-Use-Recaptcha-with-Devise)