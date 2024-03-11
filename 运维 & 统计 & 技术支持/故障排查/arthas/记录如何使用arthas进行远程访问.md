# 记录如何使用arthas进行远程访问

### 背景

arthas需要在本地进行attach, 通常情况下，开发没有权限登录服务器，如何让开发使用arthas进行远程诊断呢？

公司内部一般都有一些web管理平台，供开发者去管理自己的应用，如何把arthas集成到自己的web管理平台？

### 实现思路

在公司内部的web管理平台，基于某个主机上的某个应用有个叫开启arthas调试的按钮，点击该按钮会触发如下操作：

1. 登录到对应服务器上，基于应用名称查找对应的pid
2. 检查默认的http端口是不是有pid在监听
3. 如果该端口没有被监听，直接attach该pid之后返回
   attach命令: `sudo su - <应用启动用户> -c "java /opt/arthas/lib/3.0.5/arthas/arthas-boot.jar --target-ip <主机IP> --attach-only <应用pid>"`
4. 如果监听的pid跟应用的pid相同，直接返回
5. 如果不同，则使用命令shutdown之后，再进行attach操作
   shutdown操作：`sudo java /opt/arthas/lib/3.0.5/arthas/arthas-client.jar -c shutdown <主机IP> <http端口>`
6. attach之后，然后再通过web-console连接到该主机IP, 以及http端口就可以使用了

该实现已经被使用起来了

### 问题

在多个开发者同时针对一个主机上应用进行使用arthas调试的时候可能会出现相互shutdown的情况，这个可以通过流程规范就可以了， 或者有个选择是否把对方连接shutdown的选择；对于这种也可以维护一个端口池，同一个主机上不同应用进行attach的时候使用不同端口，这样就可以解决冲突问题

### 截图

[![image](https://user-images.githubusercontent.com/7914597/51069492-1f108980-166b-11e9-87c0-c51b1e6e37c5.png)](https://user-images.githubusercontent.com/7914597/51069492-1f108980-166b-11e9-87c0-c51b1e6e37c5.png)

[![image](https://user-images.githubusercontent.com/7914597/51069497-394a6780-166b-11e9-838b-1653bcdaa908.png)](https://user-images.githubusercontent.com/7914597/51069497-394a6780-166b-11e9-838b-1653bcdaa908.png)