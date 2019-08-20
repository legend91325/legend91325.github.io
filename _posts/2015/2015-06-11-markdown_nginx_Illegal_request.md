# 通过配置nginx  抵御不合法请求

标签（空格分隔）： nginx

---

##`ngx_http_limit_conn_module`模块
使用此模块主要用来限制每秒请求数量，至于依据什么条件限制是由我们来自定义的。
官方文档 [Module ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)
中文翻译的 [nginx限制请求数ngx_http_limit_req_module模块](http://www.ttlsa.com/nginx/nginx-limiting-the-number-of-requests-ngx_http_limit_req_module-module/)

文档讲的很详细了，大致说下：
`limit_req_zone $variable zone=name:size rate=rate;`
命令的意思是，以`$variable`变量为条件，起名为`name`,设置的存储空间大小为`size`，设置限定频率为`rate`;

我们可以设置**多个，不同条件，不同名称，不同大小的限制**。
这个定义我们是需要写在**`http`配置段中**。
在匹配的location中写上`limit_req zone=name [burst=number] [nodelay];`这里`burst`就是允许的漏桶数，当请求频率大于`rate`但是超出的数量不大于`burst`设置的数量，则nginx会将超出的请求延迟后面返回。如果请求数量超出`burst`了，则将超出部分直接返回错误码，默认`503`。至于`nodelay`就是设置是否要延迟，有它不超过`burst`的请求才延迟。
**网上大多条件都是`$binary_remote_addr`,其实我们可以根据自己的需求，来定义自身的相应条件，活学活用嘛，下面会有实例。**

---
##`ngx_http_limit_conn_module`模块
这个模块主要限制单独ip同一时间的连接数
官方文档 [Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)。
中文翻译的 [nginx限制连接数ngx_http_limit_conn_module模块](http://www.ttlsa.com/nginx/nginx-limited-connection-number-ngx_http_limit_conn_module-module/)。

各位看文档吧，我的实战中没有使用此模块。

---
##实战 阶段

好了，下面进入实战阶段：
首先我们的初始配置文件时是（不完整）：
```nginx
http {
    server {
        listen  8080 default_server;
        server_name  localhost:8080;
        
    location ~ .* {
		proxy_pass http://127.0.0.1:8080;
		proxy_set_header X-Real-IP $remote_addr;
	    }
    }
}

```
我们的需求是，有一批接口被频繁的不合法访问，我们要做限制。
第一版：限制为1s一次请求，漏桶数为5。
```nginx
http {

    limit_req_zone  $binary_remote_addr  zone=one:10m rate=1r/s;
    server {
        listen  8080 default_server;
        server_name  localhost:8080;
        
    location ~ .* {
		proxy_pass http://127.0.0.1:8080;
		proxy_set_header X-Real-IP $remote_addr;
	    }
	    
	location ^~ /interface {
		limit_req zone=one burst=5 nodelay;
		proxy_pass http://127.0.0.1:8080;
	    }
    }
}
```
>这里加了`proxy_pass http://127.0.0.1:8080;`这里配置了转发，否则匹配之后会找不到服务器的。

    但是这样会有个问题，目前我们是以ip做的限制，但是有可能网吧或者校内出口就是一个或几个ip，我们这样限制的话会把正常用户也限制到了，得不偿失。其实我们可以换一种思路来定位到单一用户，正常一个请求过来，我们都会设置携带一个关于用户的`token`信息。至于这个`token`是如何生成的，只有服务器知道，那我们加入我们的每次请求中，`header`中带有这个信息，`token`值，如果一个非法的请求可能没有这个值，即使有这个值我们也可以以`token`为条件来限制，这样更合理些。

第二版

```nginx
http {

    limit_req_zone  $http_token  zone=two:10m rate=1r/s;
    server {
        listen  8080 default_server;
        server_name  localhost:8080;
        
    location ~ .* {
		proxy_pass http://127.0.0.1:8080;
		proxy_set_header X-Real-IP $remote_addr;
	    }
	    
	location ^~ /interface {
	    if($http_token=""){
	        return 403;
	    }
		limit_req zone=two burst=5 nodelay;
		proxy_pass http://127.0.0.1:8080;
	    }
    }
}
```
在`nginx`中，使用`$http_变量名`，取的就是`header`中相应的变量。
>前方预警：我特意在这个配置中留了个坑，如果你像我这样配置的话，重启会报一个异常`nginx: [emerg] unknown directive "if($http_token" `，很奇怪是不，这个异常我花了很长时间才解决，原因是`if`和`(`中间需要个**空格**，没错，就是这个空格花了我好几个小时，血泪的教训啊，希望各位不要再重蹈覆辙。
这个问题的解决的文章：[Nginx unknown directive “if($domain”](http://stackoverflow.com/questions/20286369/nginx-unknown-directive-ifdomain)

这次的配置，多少可以限制住的，对我一个nginx的小白来说，调研一点用一点，也是不错的。

---

参考文章：
[通过nginx配置文件抵御攻击](http://drops.wooyun.org/tips/734)
[Nginx配置proxy_pass转发的/路径问题](http://wangwei007.blog.51cto.com/68019/1103734)
[nginx限制请求数ngx_http_limit_req_module模块](http://www.ttlsa.com/nginx/nginx-limiting-the-number-of-requests-ngx_http_limit_req_module-module/)
[在Nginx中记录自定义Header](http://gunner.me/archives/363)