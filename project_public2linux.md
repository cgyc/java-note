# 发布springboot项目到服务器

1. 到项目放置目录下上传jar文件：rz (选择文件)
2. 创建日志文件nohup.out，用于输出启动日志：touch nohup.out
3. 运行jar文件：nohup java -jar xxx.jar &  or 指定端口：nohup java -jar xxx.jar --server.port 8080 &
4. 查看日志文件：tail -fn 10000 nohup.out
5. 在防火墙开通端口号：vi /etc/sysconfig/iptables
6. 添加配置，保存并重启防火墙：-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT


# linux下shell脚本启动jar包工程


```linux

#!/bin/base

#jar包文件路径及名称
APP_NAME=/usr/local/src/project/echizen-api.jar
#日志文件路径及名称
LOG_FILE=/usr/local/src/project/echizen.log
#查询进程，并杀掉当前jar/java程序
pid=`ps -ef|grep $APP_NAME | grep -v grep | awk '{print $2}'`
kill -9 $pid
echo "$pid进程终止成功"

sleep 2

#判断jar包文件是否存在，如果存在则启动，并实时查看启动日志
if test -e $APP_NAME
then
echo '文件存在，开启程序...'

#启动jar包，指向日志文件，2>&1 & 表示打开或指向同一个日志文件
nohup java -jar $APP_NAME > $LOG_FILE 2>&1 &
#实时查看启动日志
tail -f $LOG_FILE
else
echo '$APP_NAME 文件不存在，请检查。'
fi
```