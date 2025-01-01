---
title: 古早博文-shell脚本---定时复制文件至目的文件夹
mathjax: true
date: 2017-10-01 12:21:15
categories: 技术
tags:
    - Linux
    - Shell
---
业务需求：定时从path1中取文件，复制到path2中

# 复制文件脚本

```bash
#! /bin/bash
path1="/newhd01/ulog_winup/"
path2="/newhd01/ulog_winup/real_for_flume/"
timelimit1="+6"
timelimit2="-17"
for i in "Search" "Download" "Browse"; do
        list_newfiles=`cd $path1$i && find -iname "*.txt" -type f -mmin $timelimit2 -a -mmin $timelimit1 | awk '{print substr($1,3)}'`
        echo ""
        echo "target folder: $path2${i}1"
        echo "Trying to copy those files in $path1$i"
        echo "$list_newfiles"
        echo ""
        OLD_IFS=$IFS
        IFS=$'\n'
        arr_newfiles=($list_newfiles)
        for s in ${arr_newfiles[@]}; do
                #echo "$s"
                isfindthisfile=`find $path2${i}1 -iname $s`
                if [ -z "$isfindthisfile" ]; then
                        echo "$path1$i/$s is not in target folder,try to copy!"
                        cp "$path1$i/$s" "$path2${i}1/$s.TMP"
                        mv "$path2${i}1/$s.TMP" "$path2${i}1/$s"
                        echo "$path1$i/$s has been moved successfully!!!"
                else
                        echo "$path1$i/$s is allready in target folder,trying to copy next !"
                fi
        done
done
IFS=$OLD_IFS
```

主要解释第7行

```bash
list_newfiles=`cd $path1$i && find -iname "*.txt" -type f -mmin $timelimit2 -a -mmin $timelimit1 | awk '{print substr($1,3)}'`
```
分3部分：
1. `cd $path1$i` 进入源文件夹目录，其实可以直接find里面加路径参数

2. `find -iname "*.txt" -type f -mmin $timelimit2 -a -mmin $timelimit1` 
[深入了解find命令](https://www.oschina.net/translate/15-practical-unix-linux-find-command-examples-part-2?print)      
找出txt格式(`-iname "*.txt"`)的文件(`-type f`)，修改时间(`-mmin`)是前timelimit1分钟 至前 timelimit2分钟 之间（`-a` 并且）的这段时间（注意这里时间的正负的含义，+表示前几分钟之前，-表示前几分钟之内）

3. `awk '{print substr($1,3)}'` 2中获取的结果是一列字符串，每一行是“./\*.txt”这种形式，利用awk处理这些字符即取第一列(`$1`)取子串第3个字符及后续
最终输出结果是n行字符（**'\n'分割**）组成的一串字符串（**并非n行的列表**）

既然上述命令输出的是一串字符串，则不可避免需要做切割。
对于一行字符串str="aaa,bbb,vvv,ccc"
直接利用${str[@]}就可以获得分割后的列表。那么，如何定义分隔符呢？
`IFS=$','`
IFS是系统自带的一个变量，储存着分隔符，默认好像是空格。可以自定义
上面脚本中就是定义了IFS为换行符。
脚本后半部分就是依次处理文件，判断目标文件夹是否已有该文件，如果没有，就复制。

# 使用crontab做定时任务
项目中源文件夹的文件是每个几分钟会增加一个，相当于上述脚本要每隔一段时间运行一次，以确保源文件夹和目标文件夹里的内容同步。
[crontab讲解1](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)
[crontab讲解2](http://blog.csdn.net/ithomer/article/details/6817019)
