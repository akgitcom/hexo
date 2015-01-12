title: rails 第二天
date: 2015-01-12 09:05:47
tags: [rails,helper,link_to,form]
---
##调用淘宝IP的API，Helper：
```ruby
module AdminsHelper
  require 'open-uri'
  require 'json'
  def hello_world(name)
    "hello #{name}"
  end
  def get_location
    ip = request.remote_ip
    url = "http://ip.taobao.com/service/getIpInfo.php?ip=" + ip
    response = Net::HTTP.get_response(URI.parse(url)).body
    response = JSON.parse(response)['data']
  end
  def get_country
    response = get_location['country']
  end
end
```
<!--more-->
###在Controller里调用Helper
在controller引入：
```ruby
include(AdminsHelper)
```

###Link_to加入html后缀，使用:format
```ruby
<%= link_to b.sbname,{:controller => "admins",:action => "show",:format=>:html,:id=>b.id} %>
```

##Form 表单提交 4.0加入了新的方法***params.require().permit()***
###定义私有方法 adminparams
```ruby
def create
	@admins = params[:admin]
	@admins[:sbpass] = Digest::MD5.hexdigest(@admins[:sbpass])
	@admins[:sblock] = 0
	@admins[:current_login_date]= Time.now.utc.to_i()
	@admins[:last_login_date] = ''
	@admins[:last_login_ip]= request.remote_ip
	@admins[:last_login_area] = get_location['country']
	@admin = Admin.new(admin_params)
	if @admin.save
		redirect_to action: 'index'
	else
		render "new"
	end
end
def show
	@admins = Admin.find(params[:id])
end
def admin_params
	params.require(:admin).permit(
		:sbname,:sbpass,:sbemail,:sblock,:current_login_date,:last_login_date,:last_login_ip,:last_login_area
	)
end
```
