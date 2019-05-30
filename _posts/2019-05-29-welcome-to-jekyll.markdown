---
layout: post
title:  "Centos7NAT模式解决网络无连接"
date:   2019-05-29 17:05:13 +0800
categories: jekyll update
---
[NAT模式下CentOS7无法连接外网以及22端口问题解决](NAT模式下CentOS7无法连接外网以及22端口问题解决)
1. 需要配置DNS解析才能够识别外部的IP域名。
首先查看是否有DNS配置
在虚拟机中输入命令:
```javascript
  cat /etc/resolv.conf
```
添加DNS配置:
```javascript
  vim  /etc/resolv.conf
  //添加如下配置即可
  nameserver 8.8.8.8
  nameserver 8.8.4.4
```
2. 打开虚拟网络编辑器
 2.1  编辑 >> 虚拟网络编辑器
#！[Alt]()




{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
