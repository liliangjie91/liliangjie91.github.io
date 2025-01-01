---
title: frp内网穿透
mathjax: true
date: 2018-10-01 13:31:26
categories: 技术
tags:
    - Linux
    - frp
    - 内网穿透
---

# 需求：
外网集群teamviwer网断，搞frp

使用：
google ip：35.194.x.161
node01:6002
web02:6001

myvultr ip： 139.180.x.224
web01:6000
bigdata01:5999

# 实践

## vps机器（远程主机，有公网ip的主机）ip：139.180.x.224
    
### 下载安装包，其实可以到 https://github.com/fatedier/frp/releases 下最新版
`wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz`

### 解压安装包
`tar -xvzf frp_0.20.0_linux_amd64.tar.gz`

### 解压后进frp_0.20.0_linux_amd64目录
看看有啥文件
```bash
ls
frpc_full.ini  frpc frpc.ini frps  frps_full.ini  frps.ini  LICENSE
#frpc frpc.ini #即客户端程序以及客户端配置文件
#frps frps.ini #即服务端程序以及服务端配置文件
```
此处，vps机器当然充当服务端的角色
即配置 frps.ini文件
`vim frps.ini`
其实里面已经有默认设置了，如下：
```yaml
[common]
bind_port = 7000
```
可以用默认设置，不用修改，然后直接启动服务端即可：
`./frps #可以加-c ./frps.ini指定配置文件，默认就是使用./frps.ini文件`
服务端搞定

## 需穿透的机器（即内网机器）
### 下载和服务端一样的安装包
`wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz`
### 解压安装包
`tar -xvzf frp_0.20.0_linux_amd64.tar.gz`
### 解压后进frp_0.20.0_linux_amd64目录
文件当然是一样的，此处，当然是使用客户端了，即./frpc
此处就需要配置客户端配置文件le
```bash
vim ./frpc.ini
#其实他也有默认配置，但是需要修改几个地方
[common]
server_addr = x.x.x.x #此处即服务端的ip
server_port = 7000  #此处是服务端配置文件里设置的bind_port 默认7000

[ssh]  #这个名字是可以随便取的，如果配置多台，可以是[ssh-01],[ssh-02]等
type = tcp
local_ip = 127.0.0.1
local_port = 22 
remote_port = 6000  #这个端口是该机器local_port映射到远程vps机器的端口
```

## 个人电脑连接内网机器
使用putty，ssh等工具，连接即可格式是：
```bash    
ssh username@server_addr -p remote_port
#username：内网机器上的某个用户名
#server_addr：vps机器的ip地址
#remote_port：内网机器frpc.ini中设置的remote_port
```

## 升级
### 设置日志：
```yaml
[common]
log_file = ./frpx.log
log_level = info
log_max_days = 3
```
### 设置kcp：
`vim frps.ini`
```bash
[common]
bind_port = 7000
# kcp needs to bind a udp port, it can be same with 'bind_port'
kcp_bind_port = 7000

# frpc.ini
[common]
server_addr = x.x.x.x
# specify the 'kcp_bind_port' in frps
server_port = 7000
protocol = kcp
```

# 参考：
https://github.com/fatedier/frp/blob/master/README_zh.md
https://sunnyrx.com/2016/10/21/simple-to-use-frp/