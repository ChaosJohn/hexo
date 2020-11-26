title: 'Hexo YAMLException: can not read a block mapping entry'
date: 2017-10-09 15:28:03
tags: [hexo]
---
![](https://avatars2.githubusercontent.com/u/6375567?v=4&s=200)

## 问题
好久没写博客，刚刚写了一篇[Nginx反向代理获取真实客户端IP](/2017/10/09/Nginx-Real-Client-IP/)，然后执行`hexo d -g`是报了错：`YAMLException: can not read a block mapping entry; a multiline key may not be an implicit key at line 4, column 1:`
![][img01]

## 查错
* 先来看看刚刚写的Markdown文件![][img02]
* 很明显，第四行只是一个分隔符，怎么可能出问题
* 谷歌一下错误，发现问题所在，其实是第三行的错误，tags:后面没有跟空格，yaml处理出错。[参考](https://github.com/fbrctr/fabricator/issues/241)

## 捐赠
最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)

[img01]: https://image.blog.chaosjohn.com/Hexo-YAMLException-can-not-read-a-block-mapping-entry/screenshot-of-problem.png
[img02]: https://image.blog.chaosjohn.com/Hexo-YAMLException-can-not-read-a-block-mapping-entry/screenshot-of-markdown.png
