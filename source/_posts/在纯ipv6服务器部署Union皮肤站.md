---
title: 在纯ipv6服务器部署MUA-Union皮肤站
date: 2024-12-10 12:51:24
tags: 
- MUA
- Union
- ipv6
- 皮肤站
- CDN
categories: 教程
---
## 前言
在ipv6普及的背景下，在纯ipv6环境下部署MUA-Union皮肤站成为可能。更重要的是纯ipv6服务器具有**价格便宜**的绝对优势😋。一些小厂商的纯ipv6机器简直白菜价，大大节省了租用大厂服务器的高额开销。
> ⚠️在此仅为举例，在下有所使用，但不对其日后服务做出保证，请自行观察决定！

例如[蓝谷科技（8465.cn）](https://8465.cn/aff/JESDDLVE)的纯ipv6服务器，【2H2G3M型】只要2.99元/月，年付28元，建个皮肤站绰绰有余。可以用优惠码ecustmc享受85折优惠。
![](https://pic2.zhimg.com/80/v2-968d1514d66f1ec1f48521b7881e7b2f.png)

## Blessing Skin安装
正常安装配置，请参考[Blessing Skin官方安装指南：https://blessing.netlify.app/](https://blessing.netlify.app/)以及[MUA文档：https://docs.mualliance.cn/zh/dev/skin](https://docs.mualliance.cn/zh/dev/skin)。

## 接入联合认证
正常步骤见[MUA文档-联合认证：https://docs.mualliance.cn/zh/dev/union/auth](https://docs.mualliance.cn/zh/dev/union/auth)的**操作指南 | 皮肤站**部分，安装好

1. 配置Union API Root代理  
由于服务器为纯ipv6，无法连接到纯v4的Union API Root（即[https://skin.mualliance.ltd/api/union](https://skin.mualliance.ltd/api/union)）。所以，必须对其进行代理。如果境外机器，可以使用Cloudflare的Warp获得v4+v6双栈网络环境，保持不变即可；境内服务器则需考虑对其进行代理。在此推荐使用支持ipv6的CDN带回源Host：`skin.mualliance.ltd`回源，相当于反向代理了。可以使用`mua.ecustvr.top`（不保证及时更换证书，用http虽然不安全，但又不是不能用...），即`http://mua.ecustvr.top/api/union`。当然，更加推荐您使用自建cdn，推荐[括彩云智能CDN](https://kuocaicdn.com/register?code=8ccmz666n3px9)和[多吉云](https://www.dogecloud.com/?iuid=6854
)。前者默认是百度cdn，每月可获得免费30G流量，不限请求数；后者默认是腾讯云cdn，每月免费20GB、200万次HTTPS请求数。注意开启CDN的ipv6访问！

2. 配置网站CDN  
由于纯ipv6网站无法被纯v4网络下访问，考虑到可能很多用户还可能处于纯v4网络下，所以建议给您的网站加上CDN。厂商同上述推荐（毕竟是免费的），源站ip处填入服务器的ipv6地址，需要注意括彩云智能CDN需要在ipv6两边加上方括号[]填入如[2001:da8:8007:1::3]，多吉云则无需。建议开启CDN的ipv6访问，SSL配置等其他配置修改建议自行百度...

3. 配置`UnionHostVerify.php`  
    - 修改网站目录下`/plugins/yggdrasil-api/src/Middleware/UnionHostVerify.php`，将`$host = gethostbyname(parse_url(option('union_api_root'), PHP_URL_HOST));`替换成`$host = gethostbyname('skin.mualliance.ltd');`。
        - ⚠️如果你使用了*括彩云智能CDN的百度线路*或者*百度智能云CDN*，前者需要发工单开启`获取真实用户IP-Client IP`，后者需要自行开启，即可将真实用户IP通过请求头 `True-Client-Ip` 传递给源站。将
            ```
                    $whip = new Whip();
                    $ip = $whip->getValidIpAddress();
            ```
            替换成
            ```
                    //根据百度CDN修改
                    $whip = new Whip(Whip::CUSTOM_HEADERS);
                    $whip->addCustomHeader('True-Client-Ip');
                    $ip = $whip->getValidIpAddress();
            ```
            否则会报错！
        - 其他CDN诸如腾讯云、阿里云、又拍云等无需修改。

至此，在纯ipv6服务器部署MUA-Union皮肤站就差不多大功告成了，Enjoy!