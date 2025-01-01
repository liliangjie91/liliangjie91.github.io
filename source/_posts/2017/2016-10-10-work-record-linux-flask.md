---
title: 古早博文-Linux下部署Flask项目——Ubuntu+Flask+Gunicorn+Supervisor+Nginx
date: 2017-10-31 23:13:23
categories: 技术
tags:
    - Linux
    - Flask
    - Web应用
---
Ubuntu16.04+Anaconda2+nginx+gunicorn+flask+supervisor

# 概述：
把Python的Flask框架开发的Web应用，在Ubuntu上用gunicorn部署起来，用supervisor实现进程守护，用nginx实现代理和负载均衡。其中的python使用的是anaconda2 这个集成包里的。并在anaconda创建的虚拟环境中运行应用。关于虚拟环境的解释见下文。

**当然，你也可以不用创建虚拟环境，直接在个人用户下执行，只需将下文中虚拟环境部分改在真实环境中执行即可。**


# 第一步：安装anaconda2 

## 下载Anaconda2的.sh安装包
Anaconda的官网在这里[Download Anaconda Now!](https://www.continuum.io/downloads)。与Python相对应，Anaconda的版本分为Anaconda2和Anaconda3，大家可以自行下载日常常用的版本，提供32位和64位下载。但下载速度太慢，建议用国内镜像：
下载地址 [Index of /anaconda/archive/](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)选择相应的版本进行下载就好。
linux 下载下来是一个*.sh文件
		
## 安装anaconda并配置源
安装Anaconda建议不要用root用户，即安装在自己的个人家目录下即可：
`bash *.sh`
安装一路按确认Yes即可。安装完成之后会在家目录下出现文件夹：
`~/Anaconda2/` （以anaconda2为例）
  注意安装过程中最后会问是否配置环境变量，记得选择yes，如果错过了，则需要**手动配置环境变量**

```bash
echo ‘export PATH="~/anaconda2/bin:$PATH"’ > ~/.bashrc
source ~/.bashrc
```
在anaconda中，conda install 命令也可以下载python库，类似于pip
首先，由于默认的源地址在国外较慢，我们需要修改其包管理镜像为国内源。
[Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)
简单来说就是在终端中运行这两个命令就好了：
```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```
执行完上述命令后，会生成~/.condarc(Linux/Mac)或C:\Users\USER_NAME\.condarc文件，记录着我们对conda的配置，直接手动创建、编辑该文件是相同的效果。
而pip的源修改如下：修改~/.pip/pip.conf，没有则创建

```yaml
[global]
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
timeout = 120
```

# 第二步：用anaconda开启一个虚拟环境
>虚拟环境是一个将不同项目所需求的依赖分别放在独立的地方的一个工具，它给这些工程创建虚拟的Python环境。它解决了“项目X依赖于版本1.x，而项目Y需要项目4.x”的两难问题，而且使你的全局site-packages目录保持干净和可管理。比如，你可以工作在一个需求Django 1.3的工程，同时维护一个需求Django 1.0的工程。

## 创建虚拟环境
python中可以用virtualenv创建虚拟环境，而我们安装的anaconda也有这个功能，创建一个名为 test1 的虚拟环境，并为这个虚拟环境**添加flask库**  

`conda create -n test1 flask`
                       

## 激活虚拟环境     
`source activate test1`
       
进入虚拟环境之后，命令行用户名前会出现一个括号，括号内是该虚拟环境的名字，表示这是在虚拟环境中


# 第三步：用flask框架构建一个简单的web应用
 新建一个目录，webtest1，内新建一个python文件webtest.py
   
直接vim编辑
```python              
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#webtest.py

from flask import Flask,request
app = Flask(__name__)

@app.route('/')
def home():
      return "home"
if __name__ == '__main__':
      app.run(debug=False)
```

此时运行该py文件，即开启了该web应用，如图所示，输入http://127.0.0.1:5000/，即可显示home界面
                    
此时，只能本地访问，如果要实现局域网访问，则需修改webtest.py：

```python
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      #webtest.py
      from flask import Flask,request
      app = Flask(__name__)
      @app.route('/')
      def home():
            return "home"
      if __name__ == '__main__':
            app.debug=False
            app.run(host='0.0.0.0')
``` 

此时，在同局域网的电脑浏览器中输入运行该代码的机器ip:5000，则显示home界面
 
![局域网浏览器浏览结果](/image/image-20161010.png)              
**截止这一步，其实我们简单完成了webapi的部署。下面的步骤都是辅助性的，目的在于提高性能，让这个web应用不容易崩。**


# 第四步：安装运行gunicorn
gunicorn是一个WSGI服务器，类似于JAVA中的tomcat。上一步中我们使用的是flask 自带的服务器，完成了 web 服务的启动。生产环境下，flask 自带的服务器，无法满足性能要求。我们这里采用 gunicorn 做 wsgi容器，用来部署 python。


## 在虚拟环境下安装gunicorn
`pip install gunicorn`
                     
               
## 用gunicorn运行web应用
  
`gunicorn tmp/webtest1/webtest:app`
webtest对应的是py文件名，app对应的是文件中Flask的实现名，这里默认开启的是8000端口。
   
![Gunicorn运行](/image/image-20161010-Gunicorn.png)          
这里使用了-b 0.0.0.0:8001 指定了端口，因为默认的8000端口已被占用

# 第五步：安装配置nginx
   [什么是nginx，nginx和gunicorn的区别](https://www.zhihu.com/question/32212996)
                    
>nginx是一个 HTTP 服务器， 它关心的是 HTTP 协议层面的传输和访问控制，所以在 Apache/Nginx 上你可以看到代理、负载均衡等功能。客户端通过 HTTP Server 访问服务器上存储的资源（HTML 文件、图片文件等等）。通过 CGI 技术，也可以将处理过的内容通过 HTTP Server 分发，但是一个 HTTP Server 始终只是把服务器上的文件如实的通过 HTTP 协议传输给客户端。
而上面所说的gunicorn，是应用服务器，是一个应用执行的容器。它首先需要支持开发语言的 Runtime（对于 Tomcat 来说就是 Java，对gunicorn来说就是python），保证应用能够在应用服务器上正常运行。其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。为了方便，应用服务器往往也会集成 HTTP Server 的功能，但是不如专业的 HTTP Server 那么强大，所以应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端。
                    作者：Leh
                    链接：https://www.zhihu.com/question/32212996/answer/55169095
                    来源：知乎
                    著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处

## 安装nginx
`sudo apt-get install nginx`
                     
nginx的配置文件在 /etc/nginx/sites-available/里，默认的是default文件，如果没有，则创建default。

```yaml                 
      server {
        listen 80;
        server_name 127.0.0.1;

        location / {
            try_files $uri @gunicorn_proxy;
        }

        location @gunicorn_proxy {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            proxy_pass http://127.0.0.1:8000;
            proxy_connect_timeout 500s;
            proxy_read_timeout 500s;
            proxy_send_timeout 500s;
        }
      }
```
## 启动nginx或重启nginx (restart)
`service nginx start`

关于nginx的详细配置可参考如下：
http://www.jianshu.com/p/fd25a9c008a0
http://www.jianshu.com/p/3e2b9964c279


# 第六步：supervisor进程守护
严重推荐这篇介绍 ：[使用 supervisor 管理进程](http://liyangliang.me/posts/2015/06/using-supervisor/)
>Supervisor (http://supervisord.org) 是一个用 Python 写的进程管理工具，可以很方便的用来启动、重启、关闭进程（不仅仅是 Python 进程）。
     除了对单个进程的控制，还可以同时启动、关闭多个进程，比如很不幸的服务器出问题导致所有应用程序都被杀死，此时可以用 supervisor 同时启动所有应用程序而不是一个一个地敲命令启动。
                    

## 安装supervisor（不在虚拟环境里）
     
`pip install supervisor`   或者使用  `sudo apt-get install supervisor`

## 配置文件 supervisor.conf
用以下命令可以把默认的配置生成到制定位置
`echo_supervisord_conf > /your_file_path/supervisord.conf`
**如果是使用apt-get安装的，那么 这个supervisord.conf文件会在 /etc/supervisor/文件夹下自动生成**
                   
```yaml
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket

[include]
files = /etc/supervisor/conf.d/*.conf
```

配置文件最后一行，设置的是要运行的进程的配置文件路径，可见只要后缀名是conf即可被识别，于是新建文件`/etc/supervisor/conf.d/webapi.conf`

```yaml
[program:webtest]
directory = /home/bigdata/Project_2017/webapi/ ; 程序的启动目录
command =/home/bigdata/anaconda2/bin/gunicorn -k gevent -t 300 run:app  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = root          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /home/bigdata/tmp/usercenter_stdout.log

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

自行修改以上配置

                    
## 设置并启动supervisord（服务端）

 首先，supervisord运行是需要有配置文件的，启动supervisord可以直接命令行运行
`supervisord`
  这时，supervisor会默认依次在这几个路径下寻找配置文件
```
  $CWD/supervisord.conf, 
  $CWD/etc/supervisord.conf, 
  /etc/supervisord.conf
```
 如果这几个路径下没有该配置文件，则启动失败。
 所以如果配置文件在其他位置例如/your_file_path/supervisord.conf ，则需要用 -c 设置
`supervisord -c /your_file_path/supervisord.conf`
                   
## 启动supervisorctl（客户端）
命令
`sudo supervisorctl -c /your_file_path/supervisord.conf`

这里客户端和服务端要使用**同一个supervisord.conf文件**，要确保两者使用的是同一个！要么都默认位置，要么都指定同一位置。
这样就可以进入supervisor客户端
[Supervisor客户端](http://upload-images.jianshu.io/upload_images/3376541-4029f2b2e0815d83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
                   
**这里的web应用是对应的/etc/supervisor/conf.d/webapi.conf文件里的配置的，我的配置文件有两个即
webapp 和  webtest**
   这样，web接口进程就被supervisor守护了，关掉终端也无所谓了。

如果要关闭应用进程，则进入supervisorctl    
里面有`stop；start；restart`等命令，后跟web应用名即可


# 参考文献：
https://zhuanlan.zhihu.com/p/21262280
http://www.jianshu.com/p/be9dd421fb8d
https://gist.github.com/binderclip/f6b6f5ed4d71fa64c7c5
http://www.cnblogs.com/Ray-liang/p/4837850.html
http://liyangliang.me/posts/2015/06/using-supervisor/
 https://www.zhihu.com/question/32212996


PS：错误之处还请指正，谢谢！