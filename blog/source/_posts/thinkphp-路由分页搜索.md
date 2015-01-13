title: thinkphp 路由分页搜索
date: 2015-01-13 13:50:14
tags: [thinkphp,路由,分页,搜索]
---
thinkphp的条件搜索分页后需要指定路由,`search`是路由的名字,可以换成任意字符,需要在route.php修改,这里是指定的参数

- i_f_searchtxt #搜索字符
- sc #select分类
- __hash__ #自带参数,必选

设置**$p->url**参数,直接拼接字符串:

```php
$p->url = "search/i_f_searchtxt/".$_GET["i_f_searchtxt"]."/sc/".$_GET["sc"]."/__hash__/".$_GET["__hash__"].'/p/';
```
**config/route.php:**
```php
$arr['URL_ROUTE_RULES']['search/:i_f_searchtxt/:sc/:p\d'] = array('Index/Search');
$arr['URL_ROUTE_RULES']['search'] = array('Index/Search');
```
> 这里的route.php是[thinkphp后台动态设置路由](http://localhost:4000/2015/01/12/thinkphp-%E5%90%8E%E5%8F%B0%E5%8A%A8%E6%80%81%E8%AE%BE%E7%BD%AE%E8%B7%AF%E7%94%B1/)文章里使用的方法自动生成的.

> **$_GET**方法还需要过滤,或者直接用thinkphp提供的`I`方法