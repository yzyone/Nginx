# 【nginx代理postgresql】 #

```
#代理pgsql数据库
stream {
   upstream pgsql {
        server 192.168.12.1:5444;
   }
    server {
        listen 25444;
        proxy_connect_timeout 30s;
        proxy_timeout 30s;
        proxy_pass pgsql;
    }
}
```