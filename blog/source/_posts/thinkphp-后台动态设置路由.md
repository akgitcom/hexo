title: thinkphp 后台动态设置路由
date: 2015-01-12 09:34:45
tags: [thinkphp,动态路由]
---
##Thinkphp 后台动态设置配置文件

> 单独加载routes.php路由文件。routes.php返回数组。

<!--more-->
##创建配置文件

```php
$Category = D('Category');
$categorylist = $Category->where('sblock=0')->select();
foreach ($categorylist as $row)
{
    $arr['URL_ROUTE_RULES'][$row[routename]]= array('Product/index','cateid='.$row['cid']);
}
$arr['URL_ROUTE_RULES']['ae/:aid\d'] = 'Article/View';
$arr['URL_ROUTE_RULES']['sv/:sid\d'] = 'Singlepage/View';
$arr['URL_ROUTE_RULES']['pt/:pid\d'] = 'Product/View';
$arr['URL_ROUTE_RULES']['fq/:fid\d'] = 'Faq/View';
$arr['URL_ROUTE_RULES']['dd/:did\d'] = 'Download/View';
$arr['URL_ROUTE_RULES']['company'] = array('Singlepage/View', 'sid=1');
$arr['URL_ROUTE_RULES']['index'] = 'Index/Index';
$arr['URL_ROUTE_RULES']['qualificatio'] = array('Singlepage/View', 'sid=2');
$arr['URL_ROUTE_RULES']['message'] = 'Guestbook/Index';
$arr['URL_ROUTE_RULES']['Factory'] = array('Singlepage/View', 'sid=4');
$arr['URL_ROUTE_RULES']['Contact'] = array('Singlepage/View', 'sid=3');
$arr['URL_ROUTE_RULES']['pc/:cateid\d'] = 'Product/Index'; //规则路由
$arr['URL_ROUTE_RULES']['pi/:cateid\d/:p\d'] = array('Product/Index');
$arr['URL_ROUTE_RULES']['pi/:p\d'] = array('Product/Index');
$arr['URL_ROUTE_RULES']['ai/:p\d'] = array('Article/Index');
$arr['URL_ROUTE_RULES']['fi/:p\d'] = array('Faq/Index');
$arr['URL_ROUTE_RULES']['di/:p\d'] = array('Download/Index');
$arr['URL_ROUTE_RULES']['s/:p\d'] = array('Product/Search');
$arr['URL_ROUTE_RULES']['s'] = array('Product/Search');
dump(F('routes',$arr, './Home/Conf/')); //重点
```

> 如果config.php有’URL_ROUTE_RULES’这个数组，会覆盖，暂时将所有路由都写到这里。。。

##将中文转换为拼音

```php
if(!empty($data['routename'])){
    $data['routename'] = $data['routename'];
}else{
    import("ORG.Util.ChinesePinyin");
    $pinyin = new ChinesePinyin();
    $data['routename'] = $pinyin->TransformWithoutTone($data['title'],'');
}
```

##config.php 配置文件，在‘URL_ROUTE_RULES’ 下面加入代码:

```php
'LOAD_EXT_CONFIG'=>'routes'
```

> 载入APP_PATH/conf/目录下的routes.php文件

