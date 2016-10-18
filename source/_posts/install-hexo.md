---
title: hexo结合github创建博客，解决多终端或跨系统同步发布
date: 2016-10-10 23:33:26
tags: hexo
---

## 安装Hexo

新建一个文件夹，这里就是我们本地的Hexo目录了，比如新建`Myblogs`文件夹,然后进入这个目录，
> cd Myblogs

然后用执行安装Hexo的命令:
> npm install hexo-cli -g

此时会出现一些WARN，不过不会影响hexo使用
> npm install hexo --save

初始化Hexo
> hexo init

安装hexo相关组件
> npm install

<!--more-->

## 开始使用hexo

先生成一个helloworld的demo网页
> hexo g

本地测试生成的网页
> hexo d

打开浏览器输入 http://localhost:4000/
可以看到生成的demo网页

新建自己的blog
> hexo n "MyBlog"

这样就新建了一个自己的blog，页面编辑` /source/_posts/ `目录下面会生成markdown文件

新建远程github仓库，仓库命名规则：你的名字.github.io

修改全局配置文件
部署到github，修改_config.yml文件中的Deployment，
```
    deploy:
        type: git
        repo: git@github.com:yourname/yourname.github.io.git
        branch: master
```
使用git的方式部署，需要执行
> npm install hexo-deployer-git --save

执行部署
> hexo d

打开自己的写的blog,打开网页
> https：//yourname.github.io

---
## 在不同终端不同系统写博客的问题

如果在其他电脑上或者系统上编辑自己的博客也很简单：（前提不同的系统或终端一定要先配置好github账户的ssh）

1. 先在远程仓库新建一个branch，比如hexo

![new branch](http://of6x0sb2r.bkt.clouddn.com/newHexoBranch.png "新建branch")

 接着将hexo的分支设为默认，
 
![default branch](http://of6x0sb2r.bkt.clouddn.com/SetDefaultBranch.png "设置默认branch")

2. 将我们的源文件上传到hexo分支

初始化本地仓库： `git init`

添加本地所有文件到仓库：`git add -A`

添加commit：`git commit -m "blog源文件"`

添加本地仓库分支hexo：`git branch hexo`

添加远程仓库：`git remote add origin git@github.com:yourname/yourname.github.io.git`

将本地仓库的源文件分支hexo强制推送到远程仓库hexo分支：`git push origin hexo -f`

![hexo branch](http://of6x0sb2r.bkt.clouddn.com/hexo_srcfiles.png "上传完成")

上传完成之后，我们就拥有了两个远程的分支：master和hexo，其中master是部署成博客的分支；hexo是我们可以clone到其他电脑或其他系统的hexo源文件的分支，而且我们已经将它设置成默认仓库；

3. 在其他电脑设备上执行clone远程仓库到本地，

> git clone git@github.com:yourname/yourname.github.io.git

进入本地仓库执行hexo安装：`npm install hexo` `npm install` ` npm install hexo-deployer-git --save`

4. 编辑本地blog之后，编辑发布博客

依次执行`git add .` ,`git commit -m "发布了啥"`， `git push origin hexo`,同步本地仓库到远程

部署发布博客: `hexo g -d`，这样就生成静态网页部署到了github中