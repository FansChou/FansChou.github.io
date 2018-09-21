---
title: Ubuntu18.04安装shadowsocks客户端
date: 2018-09-21 11:41:11
tags: 科学上网
categories: 环境安装
---

> 本文是记录了安装shadowsocks客户端的过程

<!-- more -->

# 安装shadowsocks

首先在Ubuntu下打开终端

用pip安装

```bash
sudo apt-get update
sudo apt-get install python-pip
sudo apt-get install python-setuptools m2crypto

```

接着安装shadowsocks

```bash
pip install shadowsocks

```

也可以直接不用上面的步骤，直接、

```bash
sudo apt install shadowsocks

```

当然你在安装时候肯定有提示需要安装一些依赖比如`python-setuptools m2crypto`，依照提示安装然后再安装就好

# 启动shadowsocks

安装好后，在本地我们要用到sslocal ，终端输入`sslocal --help` 可以查看帮助，通过帮助提示我们知道各个参数怎么配置，比如 sslocal -c 后面加上我们的json配置文件，或者像下面这样直接命令参数写上运行。

比如

```shell
sslocal -s 11.22.33.44 -p 8388 -k "123456" -l 1080 -t 600 -m aes-256-cfb

```

-s表示服务IP, -p指的是服务端的端口，-l是本地端口默认是1080, -k 是密码（要加""）, -t超时默认300,-m是加密方法默认aes-256-cfb。

**为了方便我推荐直接用sslcoal -c 配置文件路径 这样的方式，简单好用。**

我们可以在/home/fans/ 下新建个文件shadowsocks.json  (fans是我在我电脑上的用户名，这里路径根据自己的来)。内容是这样：

```
{
    "server":"X.X.X.X",
    "server_port":8388,
    "local_port":1080,
    "password":"123456",
    "timeout":600,
    "method":"aes-256-cfb"
}

server  你服务端的IP
servier_port  你服务端的端口
local_port  本地端口，一般默认1080
password  ss服务端设置的密码
timeout  超时设置 和服务端一样
method  加密方法 和服务端一样
```

确定上面的配置文件没有问题，然后我们就可以在终端输入

```shell
sslocal -c /home/mudao/shadowsocks.json -d -start

```

回车运行，保持上面打开的窗口不要关闭，继续下面的操作。

# 配置代理服务

在经过上面一番操作之后，我发现在linux下并不能直接通过上述设置直接翻墙，因为shawdowsocks是socks 5代理，需要客户端配合才能翻墙。

- 安装privoxy

  ```shell
  apt-get install privoxy
  
  ```

- 配置privoxy

  ```shell
  vi /etc/privoxy/config
  
  ```

  我本地实在第1337行找到修改对象的，修改端口为上面shadowsocks配置的本地端口，如下：

  ```
  forward-socks5t / 127.0.0.1:1080 .
  
  ```

  privoxy监听接口默认开启的 `localhost：8118`，也是在这个文件中，这里我没有修改。

- 启动privoxy

  - 开启privoxy 服务就行 

    ```shell
    sudo service privoxy start
    
    ```

  - 设置http 和 https 全局代理 

    ```shell
    export http_proxy='http://localhost:8118
    export https_proxy='https://localhost:8118
    
    ```

- 测试

  ```shell
  wget www.google.com
  
  ```

  如果把返回200 ，并且把google的首页下载下来了，那就是成功了

# 配置PAC

通过上面的配置，我们可以在全局走代理去访问HTTP，但是针对国内的网站，我们是不想它走代理的流量的，所以我们要通过设置PAC绕过这些国内的网站。

- 安装GenPAC

  ```shell
  sudo pip install genpac
  sudo pip install --upgrade genpac
  
  ```

- 配置GenPAC
  进入终端，cd到你希望放配置文件的目录，例如：

  ```shell
  cd /home/fans/Config
  
  ```

- 执行以下命令

  ```shell
  sudo genpac --proxy="SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
  
  ```

  注意：
  上面的127.0.0.1:1080根据本地情况填写，端口是上面shadowsocks配置的本地端口。
  如果出现下面这种报错：

  ```
  fetch gfwlist fail. online: https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt local: None
  
  ```

  那么换成执行下面的语句：

  ```shell
  sudo genpac --proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
  
  ```

- 设置系统全局代理
  进入设置：`系统设置 –> 网络 –> 网络代理`
  方法选择`自动`，
  配置url填写：`file:///home/fans/Config/autoproxy.pac`



最后通过浏览器访问[Google](www.google.com)验证代理是否配置成功。