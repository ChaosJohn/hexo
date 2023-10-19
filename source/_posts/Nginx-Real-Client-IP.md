title: Nginx反向代理获取真实客户端IP
date: 2017-10-09 10:51:57
tags: [Nginx]
---
![](https://nginx.org/nginx.png)

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Nginx-Real-Client-IP.html)

## 起因
　　此前做了一个小项目，php实现，[ip.chaosjohn.com](http://ip.chaosjohn.com), 可以用命令行来获取当前客户端ip地址，效果如下：![][img01] 
　　之前是用[CaddyServer](https://caddyserver.com)来运行这个服务，最近把其从CaddyServer迁移到nginx，很顺利的迁移成功。以学习为目的，我在该服务器上用另外一个地址ip.vultr-01.coodiin.com（以下称“反代端”）反向代理到ip.chaosjohn.com（以下称“后端”），却出现问题了，ip读取不正确，显示的是服务器的IP。Nginx配置如下：![][img02]
## 问题分析
　　出现这样的情况的原因是因为，在反向代理的过程中，对于后端而言，反代端就是客户端。
## 解决
* 要解决这个问题，则需要把真实的客户端地址告诉后端。那反代端和后端怎么沟通真实的IP地址呢？一个说另一个得听吧，两边都要设置。
* 反代端
	* proxy_set_header X-Real-IP $remote_addr; # 设置X-Real-IP为真实来源请求的IP地址
	* proxy_bind 127.0.0.1; # 规定代理请求的出口地址
* 后端 
	* set_real_ip_from 127.0.0.1/32; # 只有从这个地址来的请求才会设置真实IP，才会应用X-Real-IP这个header
* 最后的代码![][img03]

## 4. 结语
下次打算记录一下php的配置，包含Nginx和CaddyServer。最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)


[img01]: Nginx-Real-Client-IP/demo-of-ip.chaosjohn.com.png
[img02]: Nginx-Real-Client-IP/wrong-ip-when-using-proxypass.png
[img03]: Nginx-Real-Client-IP/correct-ip-when-using-proxypass.png
