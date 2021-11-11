
# nginx添加ssl证书 #

**1.为什么使用ssl证书**

数据加密传输，防止窃听
可以进行数据完整性检查，防篡改
可以进行身份验证，防冒充
提示：登陆时用HTTPS可以防止别人截获用户名和密码

**1.1 lnmp安装**

未安装lnmp环境的，可以点击安装地址，进行安装

**1.2SSL通信过程**

- 客户端向服务器发请求，请求证书
- 服务器把证书发给客户端
- 客户端对比证书，成功进入不一步，否则告警
- 服务器收到对称密钥后保存，给客户端一个应答
- 客户端接收响应，这样就完成了SSL连接，后面的通信用对称密钥加密数据传输

**1.3自签SSL证书**
 
```
#生成一个RSA私钥,1024是加密强度，一般是1024或2048 
openssl genrsa -out private.key 1024
 
#生成一个证书请求
 openssl req -new -key private.key -out cert_req.csr
 
#自己签发证书，如果要权威CA签发的话，要把cert_req.csr发给CA
openssl x509 -req -days 365 -in cert_req.csr -signkey private.key -out server_cert.crt
```

**1.4编辑配置文件nginx.conf**

```
server{
        listen 80;
        server_name lnmp;
        rewrite "^/(.*)$" https://lnmp/$1 break; #访问时做自动跳转
}
 
server {
        listen       443 ssl;
        server_name  localhost;
        ssl_certificate      /application/nginx/key/server.crt;
        ssl_certificate_key  /application/nginx/key/server.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
}
 
#重启服务
使用https://lnmp #访问网站，当前主机域名解析为lnmp
```

**1.5curl测试**

	curl http://lnmp/ 会自动跳转到https://lnmp中

**1.6 浏览器测试**

浏览器访问地址：http://lnmp/

参考链接：https://blog.csdn.net/weixin_39128119/article/details/84616260