
# nigix proxy_set_header Host $host:$server_port 与 $host:proxy_port 区别 #

nigix做反向代理  

注意  :$proxy_port  与 :$server_port 区别

$server_port ：nigix监听的端口

$proxy_port ： 服务器真正访问的端口

```
server {        listen       8888;
        server_name  192.168.1.114;        
        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        location  /a {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host:$proxy_port;
        }
        location  /b {
            proxy_pass http://192.168.1.102:8080/b;
            proxy_cookie_path /a /b;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```

-----------------------------------------------------------------------------------------------------------------

```
server {
        listen       8888;
        server_name  192.168.1.114;        
        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        location  /a {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host:$server_port;
        }
        location  /b {
            proxy_pass http://192.168.1.102:8080/b;
            proxy_cookie_path /a /b;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```

######   Nginx反向代理Tomcat访问报错400问题   #######

线上用nginx反向代理tomcat访问，配置完成后，直接访问tomcat完全正常，但是只要在nginx添加反向代理tomcat，访问nginx就会报错400。

原因和解决办法：

1）后端服务器设置有类似防盗链或者根据http请求头中的host字段来进行路由或判断功能的话，如果nginx代理层不重写请求头中的host字段，将会导致请求失败，报400错误。
解决办法：

	proxy_set_header Host $http_host;

2）nginx配置中header头部信息的host不能被配置重了。tomcat没有对headers中的host进行唯一校验。
解决办法（下面两个要去掉一个）：

	proxy_set_header Host $host;
	proxy_set_header Host $http_host;    #去掉这一行

*************** 当你发现自己的才华撑不起野心时，就请安静下来学习吧！***************