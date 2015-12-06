title: 利用hexo在GitHub上建立博客
date: 2015-12-06 11:04:36
tags:
- Hexo
categories:
- Other
---
闲着无聊在GitHub上建了个人博客~~入了坑~~，用于记录各种坑和各种搬砖心得。本博客使用的hexo版本是3.1.1，本机操作系统为Win 8.1。配置流程主要参考了[这篇](http://www.jianshu.com/p/465830080ea9)和
[另一篇](http://xinghao.me/2014/08/24/2014-08-24-blog_setup/)博客。本文先概述一下配置流程，然后记录一下配置过程中遇到的各种坑和解决方法。
<!--more-->

## 配置流程
### 本地配置
1. 安装[Node.js](https://nodejs.org/en/)和[GitHub](https://desktop.github.com/)。

2. 利用Node.js安装[Hexo](http://hexo.io/)，即在Git Shell中输入：
``` bash
$ npm install -g hexo
```

3. 初始化博客目录：
``` bash
$ hexo init blog
$ cd blog
$ npm install
```

4. 查看本地环境是否配置成功：
``` bash
$ hexo generate # 或者hexo g，用于生成静态页面
$ hexo server   # 或者hexo s，用于启动本地服务
```
此时用浏览器进入[localhost:4000](http://localhost:4000/)就能看到博客初始的页面了。

### GitHub配置
1. 在GitHub上建立仓库，名字必须为"username.github.io"（其中username替换为你的用户名）。

2. 修改blog文件夹下的_config.yml文件的deploy部分：
```
deploy:
type: git
repository: https://github.com/username/username.github.io.git
branch: master
```

3. 在Git Shell执行hexo命令：
``` bash
$ hexo g
$ hexo deploy  # 或者hexo d，用于发布页面
```

4. 在浏览器打开你的[个人页面](http://username.github.io/)查看发布的博客网站。

## 碰到的各种坑
### npm安装报错
当在执行各种npm install语句报错时，请在带管理员权限的Shell执行命令。

### hexo server命令报错
当执行hexo server命令提示找不到命令时，需要安装hexo-server：
``` bash
$ npm install hexo-server --save
```

### 本地页面错误
当访问localhost:4000页面，发现仅有一行：
> Cannot GET /

此时需要执行如下三行命令后再重新生成：
``` bash
$ npm install hexo-renderer-ejs --save
$ npm install hexo-renderer-stylus --save
$ npm install hexo-renderer-marked --save
```

### hexo deploy命令报错
#### 不是git目录
在.deploy_git目录下执行：
``` bash
$ git init
```
#### 找不到deployer
需要安装git的支持，执行如下命令：
``` bash
$ npm install hexo-deployer-git --save
```
#### git异常
执行时报如下错误：
```
INFO  Deploying: git
INFO  Clearing .deploy folder...
INFO  Copying files from public folder...
events.js:85
      throw er; // Unhandled 'error' event
            ^
Error: spawn git ENOENT
    at exports._errnoException (util.js:746:11)
    at Process.ChildProcess._handle.onexit (child_process.js:1053:32)
    at child_process.js:1144:20
    at process._tickCallback (node.js:355:11)
```
这是因为在cmd而不是Git Shell下执行命令。

#### windows下删除文件名过长的文件
由于一开始配置有问题，需要删除node_modules文件夹来重新配置，结果出现
“由于文件名太长，无法删除文件或目录”的现象，无论用命令行还是资源管理器均无法删除。最后
用过WinRAR软件的“压缩后删除原来的文件”解决了这一问题。

## 其他
### 其他常用hexo命令
除了generate、deploy和server，hexo还有如下常用命令：
``` bash
$ hexo clean  # 清理生成文件
$ hexo new "postname"  # 新建文章
$ hexo new page "pagename"  # 新建页面
```
### 日志撰写
hexo的日志基于Markdown语法（后缀名为.md），比较好的语法参考：
- [认识与入门Markdown](http://sspai.com/25137)
- [Markdown，你只需要掌握这几个](http://www.tuicool.com/articles/fmeMbqR)

常用的编辑器有[Sublime](http://www.sublimetext.com/)、[Atom](https://atom.io/)等。

### 备份源文件
当执行hexo deploy时，master分支只保留了静态页面的代码，而没有保存源代码。通常的做法是
新建一个分支，然后对源文件进行备份：
``` bash
$ git checkout -b source # 新建一个叫source的分支并切换到该分支
$ git add -A             # 添加所有文件
$ git commit -m "initial commit"  # 提交代码
$ git push origin source # 上传到GitHub
```
### 更换主题
将需要的主题主页clone到themes文件夹下即可。主题列表见该
[页面](https://github.com/hexojs/hexo/wiki/Themes)。本博客使用的是
[TKL](https://github.com/SuperKieran/TKL)主题。注意，如果要将主题文件夹也备份在
GitHub上，则必须将该主题文件夹中的.git文件夹删除。

### 配置数学公式插件
安装插件hexo-math，在Git Shell中执行：
``` bash
$ npm install hexo-math --save
```
然后在_config.yml文件中添加：
```
plugins:
- hexo-math
```
关于其使用请参照[Hexo MathJax插件](http://catx.me/2014/03/09/hexo-mathjax-plugin/)。

注：在最新版的hexo-math中，不需要执行：
``` bash
$ hexo math install
```
