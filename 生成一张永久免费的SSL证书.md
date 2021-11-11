
# 10分钟生成一张永久免费的SSL证书 #

为了安全起见，现在开发微信服务号和IOS客户端等访问服务器端都要求使用https加密传输。

SSL证书是数字证书的一种，类似于驾驶证、护照和营业执照的电子副本。因为配置在服务器上，也称为SSL服务器证书。


Let’s Encrypt 也是一个 CA 机构，但这个 CA 机构是免费的！！！也就是说签发证书不需要任何费用。

现在讲解一下，如何在centos操作系统下，获得Lets Encrypt免费的ssl证书，并在nginx里配置使用。

**1. 安装Certbot客户端**

Certbot是一个EPEL安装包，如果没有配置EPEL库，需要提前将库配置好。

运行以下命令安装Certbot:

```
$ sudo yum install certbot-nginx
```

**2.使用Certbot生成证书**

```
$ sudo certbot --authenticator standalone --installer nginx --pre-hook "nginx -s stop" --post-hook "nginx"

生成过程中需要输入域名，域名要提前进行解析。
```

**3.修改nginx配置**

certbot会在/etc/letsencrypt/live/目录下生成一个域名的目录，然后修改对应的nginx的配置。

```
ssl_certificate "/etc/letsencrypt/live/{域名}/fullchain.pem";ssl_certificate_key "/etc/letsencrypt/live/{域名}/privkey.pem";
```

**4.如何让证书永久免费呢？**

Let's Encrypt证书有效期为90天，为了保证在过期前更新证书，Certbot提供了更新证书有效期的功能。使用以下功能可以进行更新：

```
$ sudo certbot renew --dry-run

也可以使用crontab自动更新证书有效期。

0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew
```


官方链接：https://eff-certbot.readthedocs.io/en/stable/using.html#certbot-commands

参考链接：https://cloud.tencent.com/developer/news/354486