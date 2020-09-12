# linux使用

- 查看磁盘占用情况：du / -h --max-depth=1
- 获取mysql：wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
- 解压：tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
- 查看内存：df -h
- 查看当前目录下内存：du -h --max-depth=1
- 删除文件：rm -rf
- 修改或移动文件：mv mysql5.7 mysql，当前目录为修改文件名
- 创建文件夹：mkdir data
- 修改文件夹的权限：chmod -R 755 /usr/local/mysql
- 编译安装并初始化mysql：./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
- 首先检查该链接库文件有没有安装使用 命令进行核查：rpm -qa|grep libaio
- 链接库文件：yum install  libaio-devel.x86_64
- 编辑文件：vi /etc/my.cnf，按i进入编辑模式，按esc退出编辑模式，再按:wq保存退出
- 启动mysql服务：/usr/local/mysql/support-files/mysql.server start
- 添加用户组：groupadd mysql
- 添加用户：useradd -r -g mysql mysql  #useradd -r参数表示mysql用户是系统用户，不可用于登录系统
- 开启mysql服务：service mysql start
- 停止mysql服务：service mysql stop
- 登录mysql：./bin/mysql -u root -p
- 修改密码：set password=password('123456');grant all privileges on *.* to root@'%' identified by '123456';flush privileges;
- 添加远程访问：use mysql;update user set host='%' where user = 'root';flush privileges;
- 映射端口：iptables -t nat -I PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080 (外网访问80跳转内网8080)
			cd /etc/sysconfig/iptables-config
			./iptables-save
- 清空文件内容：cat /dev/null > nohup.out
- 查看占用端口：netstat -tln | grep 8072
- 查询占用端口详情：lsof -i :8072
- 杀死进程：kill -9 进程pid
- 查看软件版本：rpm -qa | grep nginx
- 