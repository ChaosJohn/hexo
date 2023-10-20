---
layout: post
title: nginx用htpasswd进行basic密码保护
date: 2023-10-20 22:56:06
tags: [nginx]
thumbnail: Protect-nginx-with-basic-auth-by-htpasswd/nginx.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Protect-nginx-with-basic-auth-by-htpasswd.md)

示例配置

```bash
server {
        listen 58185;
        listen [::]:58185;
        location / {
                auth_basic "Restricted";
                auth_basic_user_file htpasswd;

                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_ssl_verify off;
                proxy_pass http://127.0.0.1:8185;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";

                proxy_buffering off;
                client_max_body_size 0;
                proxy_read_timeout 36000s;
                proxy_redirect off;
                proxy_ssl_session_reuse off;
        }
}
```

关键指令：

```bash
auth_basic "Restricted";
auth_basic_user_file htpasswd;
```

这里面的`htpasswd`文件，可以是绝对路径，可以是nginx配置目录下的相对路径(即/etc/nginx)

htpasswd的文件内容则为一行一个用户名密码

```bash
user1:password1
user2:password2
user3:password3
```

密码是密文，可以用openssl进行生成，我们可以看一下openssl的passwd指令

```bash
$ openssl passwd -h
Usage: passwd [options] [password]

General options:
 -help               Display this summary

Input options:
 -in infile          Read passwords from file
 -noverify           Never verify when reading password from terminal
 -stdin              Read passwords from stdin

Output options:
 -quiet              No warnings
 -table              Format output as table
 -reverse            Switch table columns

Cryptographic options:
 -salt val           Use provided salt
 -6                  SHA512-based password algorithm
 -5                  SHA256-based password algorithm
 -apr1               MD5-based password algorithm, Apache variant
 -1                  MD5-based password algorithm
 -aixmd5             AIX MD5-based password algorithm

Random state options:
 -rand val           Load the given file(s) into the random number generator
 -writerand outfile  Write random data to the specified file

Provider options:
 -provider-path val  Provider load path (must be before 'provider' argument if required)
 -provider val       Provider to load (can be specified multiple times)
 -propquery val      Property query used when fetching algorithms

Parameters:
 password            Password text to digest (optional)
```

个人偏好sha256算法，即`-5`

```bash
$ openssl passwd -5 samplePassword
$5$MMjCfOFlcGmRSPlK$g8HyIjowQ6FxRy9YxKgClA161FxDkmVfdICUCOYz156
```

---

参考资料：
- [Nginx环境使用auth_basic密码保护wordpress后台登录界面](https://cloud.tencent.com/developer/article/1852440)
- [nginx-htpasswd下对网站进行密码保护](https://www.kancloud.cn/wyj0309/nginx-htpasswd-auth/533569)

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](hello-world/donate-me.png)
