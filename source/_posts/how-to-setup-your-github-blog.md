---
title: how to setup your github blog
categories: '工具与经验'
tags: [github,blog]
---
1、在github上创建一个新的repo，命名为[your_github_username].github.io

2、git clone 远程仓库到本地，本地仓库路径记为${local_path}，下文会用到。

3、搭建hexo 本地站点，参考[这篇文章](http://div.io/topic/1691)

4、生成站点内容。终端中切换到hexo本地站点根目录下，执行以下命令：

`hexo g`

5、拷贝public目录下内容到local_path目录下

`cp -R public/* ${local_path}`

6、提交本地仓库变更到远程仓库

`git add .`

`git commit -m 'commit message'`

`git push origin master`



其中第4-6步的操作可以写一个脚本自动执行。

在hexo本地站点根目录下新建一个脚本文件deploy.sh，输入以下内容：

`hexo g`
`cp -R public/* ${local_path}`
`cd ${local_path}`
`git add .`
`git commit -m 'update'`
`git push origin master`

之后每次更新blog只需要在hexo本地站点根目录下执行这个脚本文件即可。

`sh deploy.sh`

7、更换hexo主题

参考[有哪些好看的 Hexo 主题？](https://www.zhihu.com/question/24422335)

8、语言问题

在hexo本地站点的根目录下有一个_config.yml文件，可以对语言做配置。



