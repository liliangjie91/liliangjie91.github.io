---
title: 古早博文-Windows Linux 文件互传
mathjax: true
date: 2017-10-01 12:23:16
categories: 技术
tags:
    - Linux
---
# 方法1： 建立FTP服务器
[见此文章](http://www.jianshu.com/p/dfbfdbe12aae)

# 方法2： 在Windows上安装Pscp.exe

## 下载
[Putty官网](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)
  
## 使用
1.在软件所在文件夹按**shift+鼠标右键**，选择在`此处打开命令窗口`
![打开命令窗口](/image/image20161001.png)
2.在命令框中输入命令
-  把win上的local_file传到ip:/remote_dir上。使用Linux用户user1
  `pscp.exe local_file user1@ip:/remote_dir`   
- 文件夹传输
`pscp.exe -r local_folder user1@ip:/remote_dir`
- 在Linux上取文件
`pscp.exe user1@ip:/remote_file local_dir` 
- 直接在命令中带用户名和密码
 `pscp.exe -l user1 -pw ****** local_file ip:/remote_dir`   

# 方法3：建共享文件夹----Linux安装samba

## 安装
`sudo apt-get install samba`
## 设置一个文件夹用于共享并修改权限
`sudo mkdir /path/ForShare`
`sudo chmod -R 777 /path/ForShare`
## 备份并修改新的samba配置文件
`sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak`
`vim /etc/samba/smb.conf`
添加如下内容
```yaml
[share]    #此共享配置的名字
path = /path/ForShare    #共享文件夹在Linux上的位置
public = yes    
writable = yes        #可写
valid users = user1  #登录用户名，需Linux用户
create mask = 0644
force create mode = 0644
directory mask = 0755
force directory mode = 0755
available = yes
```
设置密码
`sudo touch /etc/samba/smbpasswd`
`sudo smbpasswd -a user1`
输入两次密码以确定

## 启动samba
`sudo /etc/init.d/samba restart`

## 验证
![alt text](/image/image2016100102.png)
