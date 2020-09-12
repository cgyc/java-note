# nginx的反向代理

- 命令：要在nginx.exe当前目录下执行
	* 启动：start nginx
	* 停止：nginx -s stop
	* 重启：nginx -s reload / service nginx restart
	* 查看状态：nginx -t

## 主要的配置信息

- server: 服务器的配置信息，可以有多个
	* listen: 服务器的监听端口
	* server_name: 需要监听的路径，当浏览器访问该路径时，跳转到真实的服务器路径
	* location: 本地拦截配置，可以正则匹配
		+ root: 映射的本地文件路径

eg: an.echizen.com我们没有，所以要在hosts文件里配置
/* 
server {
		
        listen       80; 
        server_name  an.echizen.com; #需要修改C:\Windows\System32\drivers\etc\hosts文件

        location / {
            #root   html;
			proxy_pass	http://localhost:8070;
			index  index.html index.htm;
        }

        #error_page  404              /404.html;

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    } 
*/

- 可以设置多个服务：

upstream server1 {
	server  localhost:8080;
}
upstream server2 {
	server  localhost:8081;
}

location / {
            #root   html;
			proxy_pass	http://server1;
			index  index.html index.htm;
        }