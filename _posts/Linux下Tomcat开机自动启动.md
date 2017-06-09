---
layout: post
title:  Linux下Tomcat开机自动启动
date:   2017-6-9 10:39:38 +0800
categories: Linux
tag: 
---

* content
{:toc}


- 第一步：我们在/etc/init.d下新建一个文件tomcat（需要在root权限下操作）

```
vi /etc/init.d/tomcat  

```
写下如下代码，tomcat自启动脚本：

``` shell
#!/bin/sh  
# chkconfig: 345 99 10  
# description: Auto-starts tomcat  
# /etc/init.d/tomcatd  
# Tomcat auto-start  
# Source function library.  
#. /etc/init.d/functions  
# source networking configuration.  
#. /etc/sysconfig/network  
RETVAL=0  
export JAVA_HOME=/usr/java/jdk1.7.0_60  
export JRE_HOME=/usr/java/jdk1.7.0_60/jre  
export CATALINA_HOME=/usr/local/tomcat  
export CATALINA_BASE=/usr/local/tomcat  
start()  
{  
        if [ -f $CATALINA_HOME/bin/startup.sh ];  
          then  
            echo $"Starting Tomcat"  
                $CATALINA_HOME/bin/startup.sh  
            RETVAL=$?  
            echo " OK"  
            return $RETVAL  
        fi  
}  
stop()  
{  
        if [ -f $CATALINA_HOME/bin/shutdown.sh ];  
          then  
            echo $"Stopping Tomcat"  
                $CATALINA_HOME/bin/shutdown.sh  
            RETVAL=$?  
            sleep 1  
            ps -fwwu root | grep tomcat|grep -v grep | grep -v PID | awk '{print $2}'|xargs kill -9  
            echo " OK"  
            # [ $RETVAL -eq 0 ] && rm -f /var/lock/...  
            return $RETVAL  
        fi  
}  
  
case "$1" in  
 start)   
        start  
        ;;  
 stop)    
        stop  
        ;;  
                                                  
 restart)  
         echo $"Restaring Tomcat"  
         $0 stop  
         sleep 1  
         $0 start  
         ;;  
 *)  
        echo $"Usage: $0 {start|stop|restart}"  
        exit 1  
        ;;  
esac  
exit $RETVAL  
```

> 这里特别提醒注意这一句ps -fwwu root | grep tomcat|grep -v grep | grep -v PID | awk '{print $2}'|xargs kill -9，熟悉Linux命令的人应该都清楚这句话的意义，这里就简单说下前半部分，查询root用户下tomcat的进程PID，个人根据实际情况修改。  

- 第二步：保存退出之后，给其增加可执行权限

``` 
chmod +x /etc/init.d/tomcat  
```

- 第三步：挂载

``` 
ln -s /etc/init.d/tomcat /etc/rc2.d/S16Tomcat  
```
- 第四步：设置脚本开机自启动

> 把这个脚本设置成系统启动时自动执行，系统关闭时自动停止，使用如下命令  


```
chkconfig --add tomcat  
```

> 添加这个脚本之后我们启动，停止，重启tomcat可以直接用命令

``` 
service tomcat start  
service tomcat stop  
service tomcat restart  
```
