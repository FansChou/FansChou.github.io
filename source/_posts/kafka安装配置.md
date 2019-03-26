---
title: kafka安装配置
date: 2018-09-25 14:26:11
tags: kafka
categories: 环境安装
---

> 本文是记录自己在使用kafka开发时初次安装时踩过的一些坑，记录下来，方便以后查看。

<!-- more -->

# 准备工作

- 安装资料下载
  - kafka安装包，来自[kafka官方下载](http://kafka.apache.org/downloads)，可以自行选择版本
    {% asset_img 20180424_01_001.png kafka官方下载 %}

# 开始安装

以下安装是在Windows系统下执行，Linux系统类似的做调整即可

## 配置修改

- 解压缩下载的安装包到本地，开始修改配置[^1]
  - `server.properties`在根目录的config文件夹下
    log.dirs=修改为你所使用系统的路径
    例如Windows下：D:/Dependency/kafka_2.12-1.1.0/logs/kafka-logs
  - `zookeeper.properties`在根目录的config文件夹下
    dataDir=修改为你所使用系统的路径
    例如Windows下：D:/Dependency/kafka_2.12-1.1.0/logs/zookeeper-logs
- 以上修改的两个文件主要是可能会引起启动问题的配置，至于端口等设置可随意调整，此处选择默认端口号即可

## 启动文件修改

- 在根目录下找到bin/windows文件夹，用文本编辑器修改`kafka-run-class.bat`文件
  {% asset_img 20180424_01_002.png kafka-run-class %}
  我从官网下载的是编译好的安装包，在这个文件里留有一些问题，由于安装包已经编译好，其实上面对文件夹很多的扫描是没有必要的，我在安装之后的启动过程中报了一下找不到主类的错误，琢磨了很久，将上面对一些无效文件夹的扫描全部删除掉，遂解决了报错的问题。但我不能完全确定是否是因为这个导致报错，如果有出现类似的错误，可以试着操作一下。
  另外一个问题就是，Windows下的环境变量配置的时候，有些目录是带有空格的，这个也会导致报错，典型的就是如果JDK安装在C盘的Program Files下时的JAVA_HOME配置
  {% asset_img 20180424_01_003.png kafka-run-class %}
  这里将%CLASSPATH%外面加上双引号，用来解决这种情况
- 这里再多说一句，早前的kafka版本需要额外下载一个zookeeper，用来运行kafka，现在新的版本已经自带了一个zookeeper的server，直接启动即可。后面在启动服务时也会提到。
  到这里，配置的问题基本解决了，下面开始启动服务

## 启动服务

此处仍以Windows下的操作为例，Linux下所不同的是执行shell脚本而非批处理文件。

首先进入到你解压的kafka根目录

1. 启动zookeeper

    ```shell
    ./bin/windows/zookeeper-server-start.bat ./config/zookeeper.properties
    ```

2. 启动kafka

    ```shell
    ./bin/windows/kafka-server-start.bat ./config/server.properties
    ```

3. 创建topic，这里命名为test

    ```shell
    ./bin/windows/kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
    ```

4. 启动生产者客户端

    ```shell
    ./bin/windows/kafka-console-producer.bat --broker-list localhost:9092 --topic test
    ```

5. 启动消费者客户端

    ```shell
    ./bin/windows/kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning This is a message
    ```

PS：最开始，我是单独下载zookeeper配置的server，在启动消费者客户端时会发生报错，但是依旧可以完成发送和订阅的操作，也把启动命令记在下面

```shell
./bin/windows/kafka-console-consumer.bat --zookeeper localhost:2181 --topic test
```

接下来就可以在第4步打开的客户端上输入信息并回车，再进入第5步打开的客户端上查看是否收到了订阅

如果可以收到信息，那么这部分的环境搭建已经完成