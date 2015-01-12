title: thinkphp ip 黑名单&白名单
date: 2015-01-11 22:09:41
tags: php
---
获取搜素引擎蜘蛛访问情况，入库统计。
<!--more-->
```php
public function is_bot()
{
    $data['robotspage'] = $_SERVER['REQUEST_URI'] ? $_SERVER['REQUEST_URI'] : '';
    $data['oldurl'] = $_SERVER['HTTP_REFERER'] ? $_SERVER['HTTP_REFERER'] : '直接访问';
    import('ORG.Net.IpLocation'); // 导入IpLocation类
    $Ip = new IpLocation('UTFWry.dat'); // 实例化类 参数表示IP地址库文件
    $data['robotsip'] = get_client_ip();
    $useragent = addslashes(strtolower($_SERVER['HTTP_USER_AGENT']));
    if (strpos($useragent, 'googlebot') !== false) {
    $bot = 'Google';
    } elseif (strpos($useragent, 'mediapartners-google') !== false) {
    $bot = 'Google Adsense';
    } elseif (strpos($useragent, 'baiduspider') !== false) {
    $bot = 'Baidu';
    } elseif (strpos($useragent, 'sogou spider') !== false) {
    $bot = 'Sogou';
    } elseif (strpos($useragent, 'sogou web') !== false) {
    $bot = 'Sogou web';
    } elseif (strpos($useragent, 'sosospider') !== false) {
    $bot = 'SOSO';
    } elseif (strpos($useragent, 'yahoo') !== false) {
    $bot = 'Yahoo';
    } elseif (strpos($useragent, 'msn') !== false) {
    $bot = 'MSN';
    } elseif (strpos($useragent, 'msnbot') !== false) {
    $bot = 'msnbot';
    } elseif (strpos($useragent, 'sohu') !== false) {
    $bot = 'Sohu';
    } elseif (strpos($useragent, 'yodaoBot') !== false) {
    $bot = 'Yodao';
    } elseif (strpos($useragent, 'twiceler') !== false) {
    $bot = 'Twiceler';
    } elseif (strpos($useragent, 'ia_archiver') !== false) {
    $bot = 'Alexa_';
    } elseif (strpos($useragent, 'iaarchiver') !== false) {
    $bot = 'Alexa';
    } elseif (strpos($useragent, 'slurp') !== false) {
    $bot = '雅虎';
    } elseif (strpos($useragent, 'bot') !== false) {
    $bot = '未知'.$useragent;
    }
    $data['robotsname'] = $bot;
    $data['robotsarea'] = serialize($Ip->getlocation($data['robotsip']));
    $data['create_time'] = time();
    if (isset($bot)) //如果是蜘蛛就存到数据库
    {
        //$Jrobot->create();非表单无隐藏域,无法使用create()方法
        $Jrobot = D("Robot");
        $lastInsId = $Jrobot->add($data);
    }
```