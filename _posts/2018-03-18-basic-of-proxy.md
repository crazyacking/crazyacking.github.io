---
title: about proxy
description: Ceph is a unified, distributed storage system designed for excellent performance, reliability and scalability.
categories:
 - proxy
tags: proxy
photos:
- 
---

常用的proxy

- nginx
- haproxy
- tinyproxy
- squid

推荐nginx，参考：https://blog.csdn.net/qq_31666147/article/details/53057287

如果不想写到 ngnix.conf 中，那么可以在相同的目录下建立另外一个文件夹存放单独的文件，比如新建一个  proxy 的子目录，然后再在里面新建文件 prox.conf ，然后添加如下内容：

```
server {
	resolver	8.8.8.8;
	access_log	off;
	listen	8088;
	location / {
		proxy_pass	$scheme://$host$request_uri;
		proxy_set_header Host $http_host;
		proxy_buffers	256 4k;
		proxy_max_temp_file_size	0k;
	}
}
```

接着在 ngnix.conf 的 http 块中添加：

include proxy/*.conf;


将上面文件的配置包含进来。

上面使用谷歌 DNS 8.8.8.8，你可能在本地也需要使用这个 DNS，否则可能会出现 502 的错误。不然可以配置 resolver 地址为你 ISP 分配的 DNS 地址。

配置完后，重启一下 nginx 或 reload 一下即可。

注意：由于 HTTP 代理和 VPN 不一样，后者加密，而前者不加密，所以 HTTP 代理是不能用来 FQ 的。


下面是类似配置内容：

```
server {
    resolver 202.106.0.20;
    resolver_timeout 5s;
    listen 81;
    location / {
        proxy_pass scheme://host$request_uri;
        proxy_set_header Host $http_host;
        proxy_buffers 256 4k;
        proxy_max_temp_file_size 0;
        proxy_connect_timeout 30;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 301 1h;
        proxy_cache_valid any 1m;
    }
}
```


参数解析：
1，配置 DNS 解析 IP 地址，比如 北京dns，以及超时时间（5秒）。
resolver 202.106.0.20;
resolver_timeout 5s;

注意项
1. 不能有hostname
2. 必须有resolver, 即dns，即上面的x.x.x.x，换成你们的DNS服务器ip即可
3. 配置正向代理参数，均是由 Nginx 变量组成。其中 proxy_set_header 部分的配置，是为了解决如果 URL 中带 "."（点）后 Nginx 503 错误。

`proxy_pass $scheme://$host$request_uri`; `$http_host`和`$request_uri`是nginx系统变量，不要想着替换他们，保持原样就OK。
proxy_set_header Host $http_host;

4. 配置缓存大小，关闭磁盘缓存读写减少I/O，以及代理连接超时时间。

```
proxy_buffers 256 4k;
proxy_max_temp_file_size 0;
proxy_connect_timeout 30;
```

5. 配置代理服务器 Http 状态缓存时间。

```
proxy_cache_valid 200 302 10m;
proxy_cache_valid 301 1h;
proxy_cache_valid any 1m;
```

三、不支持代理 Https 网站
因为 Nginx 不支持 CONNECT，所以无法正向代理 Https 网站（网上银行，Gmail）。