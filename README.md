<p align="center">
    <img src="https://raw.githubusercontent.com/dreamans/syncd/master/resource/logo.png" alt="Syncd">
</p>
<h3 align="center">Syncd - 自动化部署工具</h3>
<p align="center">
  <a href="https://travis-ci.org/dreamans/syncd"><img src="https://travis-ci.org/dreamans/syncd.svg?branch=master" /></a>
</p>

syncd是一款开源的代码部署工具，它具有简单、高效、易用等特点，可以提高团队的工作效率.

**目前只支持类Linux系统.**

## 目录
- [特性](#特性)
- [原理](#原理)
- [安装](#安装)
- [使用](#使用)
- [Todo](#Todo)
- [关于](#关于)
- [授权](#授权)

## 特性

- Go语言开发，编译简单、运行高效
- Web界面访问，交互友好
- 灵活的角色权限配置
- 支持Git仓库
- 分支、tag上线
- 部署hook支持
- 完善的上线工作流
- 邮件通知机制

## 原理

## 安装

### 准备工作

- Go 

推荐Go1.10以上版本, 用来编译源代码

- Nginx

Web服务依赖Nginx

- MySQL

系统依赖Mysql存储持久化数据, 推荐版本 `Mysql 5.7`

- Linux + Bash

系统会使用到 `git`, `ssh`, `scp` 等命令，所以目前只推荐在Linux上使用, 并且需要提前安装或更新这些命令道最新版本

- 秘钥配置

由于部署服务器(Syncd服务所在的服务器)与生产服务器(代码部署目标机)之间通过ssh协议通信，所以需要将部署机的公钥 (一般在这里: `~/.ssh/id_rsa.pub`)加入到生产机的信任列表中(一般在这里 `~/.ssh/authorized_keys`)

可使用 `ssh-copy-id` 命令添加，或手动拷贝. 拷贝后不要忘记进行测试连通性 `ssh {生产机用户名}@{生产机地址}` 

最后建议将以下配置加入到部署服务器ssh配置`/etc/ssh/ssh_config`中，关闭公钥摘要的显示提示，防止后台脚本运行失败

```
Host *
    StrictHostKeyChecking no
```

### 安装

- 运行以下命令

```
curl https://raw.githubusercontent.com/dreamans/syncd/master/install.sh |bash
```

当前路径中若生成 `syncd-deploy` 或者 `syncd-deploy-xxx` 目录则表明安装成功

> 生成的 `syncd-deploy` 目录可拷贝或移动到你想要的地方，但不要试图将此目录拷贝到其他服务器上运行，会造成不可预料的结果.

- 数据库依赖

你需要将 `github.com/dreamans/syncd/syncd.sql` 数据表结构和数据导入到MySQL数据库中

- 修改配置文件

修改 `syncd-deploy/etc/syncd.ini` 中相关配置信息, 具体配置描述可参考注释

- 启动服务

```
cd syncd-deploy

./bin/syncd -c ./etc/syncd.ini
```

- 添加Nginx配置

```
upstream syncdServer {
    server 127.0.0.1:8868 weight=1;
}
server {
    listen       80;
    server_name  deploy.syncd.cc; # 此处替换成你的真实域名
    access_log   logs/deploy.syncd.cc.log;

    location / {
        try_files $uri $uri/ /index.html;
        root /path/syncd-deploy/public; # 此处/path请替换成真实路径
        index index.html index.htm;
    }

    location ^~ /api/ {
        proxy_pass          http://syncdServer;
        proxy_set_header    X-Forwarded-Host $host:$server_port;
        proxy_set_header    X-Real-IP     $remote_addr;
        proxy_set_header    Origin        $host:$server_port;
        proxy_set_header    Referer       $host:$server_port;
    }
}
```

重启nginx服务

### 修改hosts

若域名未解析，可修改hosts进行临时解析

```
sudo vim /etc/hosts

127.0.0.1  deploy.syncd.cc;
```

### 安装完成

打开浏览器，访问 `http://deploy.syncd.cc`

初始账号: 

```
用户名: syncd
邮箱: syncd@syncd.cc
密码: syncd.cc
```

## 使用


## Todo


## 关于


## 授权

本项目采用 MIT 开源授权许可证，完整的授权说明已放置在 LICENSE 文件中
