---
title: Linux 服务器环境配置记录（一）
date: "2025-09-30 16:11:06"
draft: true
---

<font size=4>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;关于在服务器上搭建环境的教程网上已经有很多很多，这篇文章只是自己在配置过程中的记录，以防时间久了忘记，再需要配的时候还是狼狈地到处找教程问ai。
## 1. ssh 连接远程服务器
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;获得服务器的访问权限之后一般会拿到用户名，密码与远程主机的ip或是写好的配置文件。

### 1.1 配置文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ssh的配置文件`config`放在用户目录的`.ssh`文件夹下，我们需要根据信息编辑这个文件（没有的话就新建）
```shell
cd ~
mkdir .ssh
touch config
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在`config`文件中添加以下信息：

```sh
Host <your_custom_host_name> # 名字可以自定义，作为服务器标识
  Hostname <ip_address> 
  Port <port> # 如果没有提供，端口默认为22，可以省略这一行
  User <your_user_name>
```

### 1.2 第一次登录
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在本机使用如下命令
```shell
ssh <your_user_name>@<ip_address> # ip_address也可以是在配置文件中自定义的 Host 名
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后系统会提示未识别主机名，确认连接输入`yes`即可。（第一次连接大概率是不会有人蹲着黑你的）之后输入密码即可登录。

### 1.3 密码修改
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们拿到的一般是临时密码，所以登录后会马上要求修改密码，按照要求修改即可。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
