---
layout: post
title:  Problem with the SSL CA cert (path access rights)
date:   2018-01-23
tags: [php,curl,ssl,ca]
---

>   不积跬步，无以至千里；不积小流，无以成江海。
>
>   ——荀子

# 解决方案

此问题产生的原因是ca证书过期或无效造成的。重启php-fpm

参考1：[https://github.com/archlinuxfr/yaourt/issues/287#issuecomment-292816272]()

参考2：[https://www.tuicool.com/articles/ee2iUju]()

```shell
curl-config --ca
```



