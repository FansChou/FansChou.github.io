---
title: Hexo安装部署与同步
date: 2018-08-16 22:36:53
tags: Hexo
categories: 环境安装
---

> 本文介绍了如何安装和部署 Hexo 到 Github Pages 上，并且利用 Github 进行静态网页和博客书写环境的同步。

<!-- more -->

# 前言

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。 

为了适应在多个设备切换进行博客书写的要求，利用 Github 仓库的分支分别存储 Hexo 源文件和发布出去的静态网页，这样只需在不同设备上将源文件分支同步之后，即可直接进行修改源文件进行发布，同时修改的源文件也可以提交到仓库的分支上。

# 安装

## 安装前提

安装 Hexo 相当简单。然而在安装前，你必须检查电脑中是否已安装下列应用程序： 

- [Node.js](https://nodejs.org/zh-cn/)
- [Git](https://git-scm.com/)

如果你的电脑中已经安装上述必备程序，那么接下来只需要使用 npm 即可完成 Hexo 的安装。

``` bash
npm install -g hexo

```

如果你的电脑中尚未安装所需要的程序，请根据以下安装指示完成安装。 

## 安装 Git

- Windows：点击上面的链接去官网下载后进行安装，记住在安装的选项中勾选**Add to PATH** 

- Linux (Ubuntu, Debian)：`sudo apt-get install git-core`
- Linux (Fedora, Red Hat, CentOS)：`sudo yum install git-core`

## 安装 Node.js

可以直接下载 [安装程序](https://nodejs.org/zh-cn/) 来安装。 

对于在Windows上安装的用户来说，和Git一样记住在安装的选项中勾选**Add to PATH**选项。 

## 安装 Hexo

所有必备的应用程序安装完成后，即可使用 npm 安装 Hexo。

```bash
npm install -g hexo

```

然后输入命令`hexo -v`，回车后输出hexo的版本号即为安装成功 

# 部署

以下内容皆是在Windows环境下的操作。

## Github准备

博客是在[Github](https://github.com/)上托管维护的，所以当然需要一个Github的账号了。我的Github账号是FansChou，所以我创建的仓库名字就是`FansChou.github.io`。这里账号的大小写有没有影响，我没有试验，根据我的观察应该是不影响的。

建仓库时，名字固定为 `你的账号.github.io` 这样的形式，至于为什么是这种形式，是因为 Github Pages 提供了2种类型的页面

- 一种是 `Project Pages` ，用来作为代码仓库的一个静态页面说明，这个是每个代码仓库都是可以有一个的
- 一种是 `User Pages` ，是提供给整个账号的，每个账号有且只有一个，是可以直接通过`你的账号.github.io`去这个地址去访问网页的，所以必须写成上面的那种格式。我们要部署的就是后一种的 `User Pages`

除此之外， 要想使用Git进行与远程仓库的交互，还需要一些配置，这些知识可以去网上搜索学习，这里只列出我们需要的。

1. 首先，设定本地的邮箱和用户名

   ```bash
   git config --global user.email xxx@gmail.com
   git config --global user.name xxx
   
   ```

2. 然后生成SSH公钥

   ```bash
   cd c:/user/fans/.ssh
   ssh-keygen -t rsa -C xxx@gmail.com
   
   ```

   这里需要说明两件事：首先生成公钥的时候，最好进入当前用户的 `.ssh` 目录下，再去生成，第一次试的时候，在其他地方生成的，最后进行同步的时候找不到公钥；另外就是最后的公钥名称要以 `id_` 开头，这个比较奇怪，最开始我起了一个hexo的名字，同步的时候同样报找不到公钥，也是很郁闷，最后还是改成了 `id_rsa` ，没什么特殊要求的话，还是不作妖了，哈哈~

3. 添加SSH公钥到Github账户

   将 `.ssh` 目录下的 `id_rsa.pub` 文件用 VS CODE 打开，复制里面的内容。登陆Github，选择settings － SSH keys  － add ssh keys，然后把复制的内容全部粘贴进去即可

## 初始化

经过以上的步骤，准备工作算是做好了，接下来就是初始化一个Hexo环境。

因为要使用 Github 进行同步，所以接下来先将建好的仓库clone一份到本地。找到之前建好的仓库地址，进行clone

```bash
git clone git@github.com:FansChou/FansChou.github.io.git

```

clone的地方会有一个文件夹生成，名字和仓库名一样 `FansChou.github.io` 。

然后，使用 Hexo 进行初始化，在你本地新建一个文件夹，进入之后，右键选择 `Git Bash Here` 进入命令行：

```bash
hexo init

```

将刚刚生成的文件拷贝至从仓库clone生成的文件夹 `FansChou.github.io` 里面。

这里之所以不直接在 `FansChou.github.io` 生成hexo源文件，是因为在执行hexo命令的时候要求所在目录是一个空文件夹。

最后我们生成静态文件并在本地启动：

```bash
hexo g
hexo s

```

这个时候我们进入浏览器访问 http://localhost:4000 就可以进入hexo主页面了，此时的首页应该是一个Helloworld页面。

## 同步博客源文件

> 如果你没有使用Github进行源文件同步的打算，那么请直接进入下一节进行发布吧！

这个时候本地已经可以预览到博客的界面了，接下来就是利用 Github 去同步 Hexo 的源文件。这里的做法是建立2个分支，一个是master，一个是source，master分支用来发布生成的静态网页，source则用来同步 Hexo 的源文件。

建好仓库后默认会有一个master分支，我们先创建一个source分支。

首先进入`FansChou.github.io`文件夹，右键选择 `Git Bash Here` 进入命令行，创建并切换到source分支：

```bash
git checkout -b source

```

将新分支推送给Github：

``` bash
git push origin source

```

这个时候登录Github去看你的仓库，会发现多了一个source的分支，建议把它设置为默认分支，因为后续的Git提交都是针对这个分支的。master分支作为静态网页的存放地点，一般不会直接用Git命令去提交，而是通过hexo插件去一次性部署提交。

然后我们把本地的源文件全部提交进仓库的source分支，在执行Git命令之前，请务必确认本地处于source分支下，

``` bash
git add .
git commit -m "首次提交" #提交注释
git push origin source

```

现在远程仓库的source分支上就有了你的源文件，当你之后需要切换另一个设备进行博客撰写的时候，可以直接由仓库进行clone。

## 发布博客

接下来就是将发布博客到 Github Pages 上。

发布之前先修改 `FansChou.github.io/_config.yml`文件中的`deploy`标签：

```yaml
deploy:
    type: git
    repository: git@github.com:FansChou/FansChou.github.io.git
    branch: master
    
```

repository换成你自己的仓库地址，branch默认是master，然后在`Git Bash Here`命令行里面安装deploy插件：

```bash
npm install hexo-deployer-git --save

```

进行generate生成静态文件并提交：

``` bash
hexo deploy -g

```

提交完成之后就可以直接用`你的账号.github.io`去访问你的博客网站啦！

# 撰写博客

## 新建文章

这里只简单介绍一下如何新建一篇文章，具体的一些使用，可以直接去[Hexo的官方文档](https://hexo.io/zh-cn/docs/)去学习。

执行新建文章命令：

```bash
hexo n "文章名称"

```

这个时候在`FansChou.github.io/source/_posts`就会有一个`文章名称.md`的文件，直接进行编辑，然后重新generate发布即可。