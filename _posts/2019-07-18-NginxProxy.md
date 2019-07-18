---
layout: post
title: Nginx的反向代理配置
date: 2019-07-18
tag: Nginx
---

### 什么是反向代理和正向代理

**正向代理**指的是，一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。
![正向代理](/images/posts/nginx/1563433893.jpg)

**反向代理**是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。
当我们有一个服务器集群，并且服务器集群中的每台服务器的内容一样的时候，同样我们要直接从个人电脑访问到服务器集群服务器的时候无法访问，必须通过第三方服务器才能访问集群
这个时候，我们通过第三方服务器访问服务器集群的内容，但是我们并不知道是哪一台服务器提供的内容，此种代理方式称为反向代理
![反向代理](/images/posts/nginx/156343913.jpg)


**反向代理的配置：**
```javascript
//修改nginx.conf

http {
    resolve 127.0.0.1 valid=5 ipv6=off;
    //resolve 监视与服务器域名对应的IP地址的更改，并自动修改上游配置，而无需重新启动nginx  
    //upstream块定义了NGINX Plus负载均衡流量的服务器。
    //上游后端
    //upstream中配置域名只会在nginx启动时解析一次，然后就一直用这个ip
    upstream my_upstream {
        server server1.example.com weight=number max_conns=number;
        /*weight=number 设置权重
          max_conns=number 限制与number代理服务器的同时活动连接的最大值
          max_fails=number 默认值为1
          max_fails=number 设置在fail_timeout 参数设置的持续时间内发生的与服务器通信的不成功尝试次数，以考虑服务器在fail_timeout参数设置的持续时间内不可用 。
          fail_timeout=time 指定数量的不成功尝试与服务器通信的时间应该考虑服务器不可用; 默认时间为10s
          backup 将服务器标记为备份服务器。当主服务器不可用时，它将被传递请求。
          down 将服务器标记为永久不可用。
          
        */
        server server2.example.com;
        ip_hash;
        //当用户第一次访问到其中一台服务器后，下次再访问的时候就直接访问该台服务器就好了，不用总变化了就发挥了ip_hash的威力了
        sticky cookie srv_id expires=1h domain=.example.com path=/;
        //可以识别用户会话，并将客户端会话中的所有请求发送到同一个上游服务器。
    }

    server {
        listen 80;
        location / {
            proxy_set_header Host $host;
            proxy_pass http://my_upstream;
            //nginx每五秒发送一次/test.php请求
            health_check interval=5s uri=/test.php match=statusok;
            proxy_set_header Host $host;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_connect_timeout 60;
            proxy_read_timeout 600;
            proxy_send_timeout 600;

        }
    }
    //match块定义响应必须满足的条件，以使上游服务器被认为是健康的 - 在这种情况下，状态代码200 OK以及包含文本的响应主体。ServerN is alive
    match statusok {
        # Used for /test.php health check
        status 200;
        header Content-Type = text/html;
        body ~ "Server[0-9]+ is alive";
    }
}

```
