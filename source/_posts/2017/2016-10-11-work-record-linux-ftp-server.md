---
title: 古早博文-Ubuntu FTP服务器搭建，配置修改，所遇问题解决
date: 2017-10-31 23:20:26
categories: 技术
tags:
    - Linux
    - FTP
---
# 1.安装
` sudo apt-get install vsftpd `
这样安装后，配置文件在 `/etc` 下即： `/etc/vsftpd.conf`
备份默认配置
`sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak`
下面就是修改配置文件了，此时，我们需要明确的，就是需求，我们的需求如下：
>1，一台win服务器定时（每两分钟）向一台Ubuntu服务器（本机器）发送一个文档。
2，该win机只能访问固定目录，不能全局访问
3，该win机对应用户名不能ssh登录Ubuntu服务器
4，另创建一个可以全局访问的用户
5，为Ubuntu服务器上的几个用户开通ftp权限

注意，最终解决的方法可能并非最优方法。

# 2.配置文件修改/etc/vsftpd.conf
- 2.1默认配置
[官方解释](https://security.appspot.com/vsftpd/vsftpd_conf.html)
[中文参考资料](http://os.51cto.com/art/201008/221842_all.htm)
- 2.2重点配置项
  - **anonymous_enable=YES/NO**
是否允许匿名登录ftp。本项目是私有项目，故设为**NO**
  - **write_enable=YES/NO**
是否允许写入。例如添加文件等。本项目为**YES**
  - **local_enable=YES/NO**
是否允许本地用户登录。本项目为**YES**
  - **local_umask=022**
默认掩码：设置用户创建的文件的默认权限。同时ftp禁止直接创建一个可执行的文件。umask码与chmod时的码互补。即若umask=022，实际权限为777-022=755。但是由于ftp又有上述的禁止条件，所以刚才的算法对文件夹适用，对于文件可以使用666-022=644。具体参考[此链接](http://blog.sina.com.cn/s/blog_49fd52cf0100nekk.html) 。本项目设置为**022**
  - **chroot_local_user=YES/NO**
是否限制用户在其目录下。YES为限制，即用户只能在自己的目录下操作，无法到其他用户目录。本项目为**YES**
  - **chroot_list_enable=YES/NO**
是否启用一个列表，这个列表由*chroot_list_file=/path/chroot_list*来指定。这个列表里一行一个用户名。
    - 如果此项设置为YES，即启用列表则：
若*chroot_local_user=YES*，那么这个列表里的用户就**可以跳出**目录限制而有权限到其他用户的路径。
若*chroot_local_user=NO*，那么这个列表里的用户就**被限制**在了自己的目录而无法到其他用户路径。
    - 如果此项设置为NO，则不启用列表，直接按*chroot_local_user*的设置来执行。

    详细参考[此链接](http://blog.csdn.net/bluishglc/article/details/42398811)

  - **allow_writeable_chroot=YES**
参考4.1
  - **/etc/ftpusers** 
这是一个文件，是个黑名单。里面用户不能登录ftp
   - **userlist_file=/etc/vsftpd.user_list**
这也是指定了一个用户列表（一行一个）。这个列表会根据其他配置而变成**白名单** 或 **黑名单**：
其他配置即：**userlist_enable 和 userlist_deny**。
  - **userlist_enable=YES/NO**
是否启用userlist（即userlist_file指定的列表）。
    - 当设置为YES时，这个列表的用户能不能登录取决于userlist_deny的值。
    - 当设置为NO时，除了/etc/ftpusers中的用户，其他系统用户都可以登录。

    默认`(即改配置被注释或不出现在vsftpd.conf文件中时)`为NO
  - **userlist_deny=YES/NO**
是否把userlist设置为黑名单。YES为设置为黑名单；NO为白名单。
**注意！！**此设置仅当**userlist_enable=YES**时才会起作用。
默认为YES。即如果只设置userlist_enable=YES，而不设置此项，则userlist里的用户是登录不了的。


# 3.结果展示
[登录过程](http://upload-images.jianshu.io/upload_images/3376541-f15abfe0213f6d45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[登陆成功](http://upload-images.jianshu.io/upload_images/3376541-7714ebc9ca5cadb1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[网页登录](http://upload-images.jianshu.io/upload_images/3376541-2063cf64d6e1f47a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 4.所遇问题
- ##### 4.1. userlist中用户登录时报错：500 OOPS: vsftpd: refusing to run with writable root inside chroot()
解决方法[见此处](https://linux-tips.com/t/500-oops-vsftpd-refusing-to-run-with-writable-root-inside-chroot/249)
>**If you're using vsftpd with chroot local user option and write enable like this:**
`write_enable=YES`
`chroot_local_user=YES`
**and you're getting following error when you log-in through ftp:**
`500 OOPS: vsftpd: refusing to run with writable root inside chroot()`
**you can fix the problem with adding following line to /etc/vsftpd.conf and restart the vsftpd service:**
`allow_writeable_chroot=YES`

这是因为如果配置中开启了chroot来控制用户路径（*`chroot_local_user=YES` 说明用户无法跳出自己的初始根目录而去访问他人的目录*），则用户不能再具有在该用户根目录下的写的权限。所以，要添加配置项**allow_writeable_chroot=YES**以开启根目录（*非系统根目录，而是用户登录后所在的最上级目录*）下写权限。

- ##### 4.2. userlist中nologin用户登录时报错：530 Login incorrect.
解决方法[见此处](https://serverfault.com/questions/358324/ftp-doesnt-allow-usr-sbin-nologin-user)
>If you are not using PAM, then vsftpd will do its own check for a valid
user shell in /etc/shells. You may need to disable this if you use an invalid
shell to disable logins other than FTP logins. Put check_shell=NO in your
/etc/vsftpd.conf.

**或者**
>Look at **check_shell** in vsftpd.conf:
`Note! This option only has an effect for non-PAM builds of vsftpd.`
`If disabled, vsftpd will not check /etc/shells for a valid user`
`shell for local logins.`
`Default: YES`
**You can add `'/usr/sbin/nologin'` to `/etc/shells`. Simple and easy solution.**
Another one is to change vsftpd.conf/PAM configuration.
Comment out this "auth ..." line in PAM case:
`$ grep shells /etc/pam.d/vsftpd`
`auth    required        pam_shells.so`

原因是vsftpd配置文件的check_shell默认是开启的
当你的用户当初创建时是用的`-s /bin/false` or `-s bin/nologin`
ftp会拿这个和`/etc/shells`文件比对，如果不包含，则报错。所以，
你可以把check_shell设置为NO
或者把`bin/nologin`添加进`/etc/shells`来解决此问题。
又或者，找到vsftpd对应的pam文件`/etc/pam.d/vsftpd`，把`auth    required        pam_shells.so`这一行注释掉。
