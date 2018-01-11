---
layout: post
title:  "thinkphp中对ip的限制"
date:   2017-03-15 21:48:40 +0800
tags: 杂谈
---

最近做得一个小活动中，需要对访问者的IP来进行限制，已达到一些营销目的，本来我是想通过TP中的IpLocation类结合IP库来进行判断，但是在找IP库的时候，找了好久没有找到，却无意中发现了淘宝的查询ip归属地的一个接口，非常简单，用法如下：

{% highlight php %}
$clientIP = getIPaddress();//获取客户端的IP地址
$taobaoIP = 'http://ip.taobao.com/service/getIpInfo.php?ip='.$clientIP;//把IP地址赋给淘宝的接口
$IPinfo = (array)json_decode(file_get_contents($taobaoIP));//接收返回信息
{% endhighlight %}

如果成功的话会返回如下格式：

{% highlight php %}
Array
(
    [code] => 0
    [data] => stdClass Object
        (
            [country] => 中国
            [country_id] => CN
            [area] => 东北
            [area_id] => 200000
            [region] => 辽宁省
            [region_id] => 210000
            [city] => 沈阳市
            [city_id] => 210100
            [county] => 
            [county_id] => -1
            [isp] => 联通
            [isp_id] => 100026
            [ip] => 175.168.27.80
        )

)
{% endhighlight %}

可以看到很多的信息，根据想限制的地区来对字段进行判断就可以达到限制IP的目的了~！

但是可能会发现上面 getIPaddress() 方法中不是TP中获取IP的方法，TP中自带获取IP的方式是get_client_ip()，本来我也想用这个方法，但是发现在移动端的时候获取的ip是非常不准确的，会飘忽不定，于是另一种方式来获取ip

{% highlight php %}
function getIPaddress(){
    $IPaddress='';
    if (isset($_SERVER)){
        if (isset($_SERVER["HTTP_X_FORWARDED_FOR"])){
            $IPaddress = $_SERVER["HTTP_X_FORWARDED_FOR"];
        } else if (isset($_SERVER["HTTP_CLIENT_IP"])) {
            $IPaddress = $_SERVER["HTTP_CLIENT_IP"];
        } else {
            $IPaddress = $_SERVER["REMOTE_ADDR"];
        }
    } else {
        if (getenv("HTTP_X_FORWARDED_FOR")){
            $IPaddress = getenv("HTTP_X_FORWARDED_FOR");
        } else if (getenv("HTTP_CLIENT_IP")) {
            $IPaddress = getenv("HTTP_CLIENT_IP");
        } else {
            $IPaddress = getenv("REMOTE_ADDR");
        }
    }
    return $IPaddress;
}
{% endhighlight %}

移动端通过此方法获取的ip是相对比较准确的，可以进行正确的判断。

通过以上的方法就可以实现对ip的限制，基本上没什么难度，非常简单。。。
