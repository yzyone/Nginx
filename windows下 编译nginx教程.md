
# windows下 编译nginx教程 #

1、安装visual studio 2017

2、安装msys2 `http://repo.msys2.org/distrib/x86_64/msys2-x86_64-20180531.exe`

3、下载nginx最新源代码zip包 `http://hg.nginx.org/nginx.org`

4、下载openssl，pcre，zlib源码,解压至nginx源码根目录：

       https://www.openssl.org/source/openssl-1.1.0i.tar.gz
       https://ftp.pcre.org/pub/pcre/pcre-8.42.zip
       http://www.zlib.net/zlib-1.2.11.tar.gz

5、打开vs2017命令行

VS 2017的开发人员命令提示符 

6、在命令行中启动msys2

    msys2_shell.cmd

7、在msys2控制台中输入

    echo $INCLUDE
    echo $LIB

查看环境变量是否正常



8、下载安装perl和nasm

```
https://www.nasm.us/pub/nasm/releasebuilds/2.14.02/win64/nasm-2.14.02-win64.zip

https://downloads.activestate.com/ActivePerl/releases/5.26.3.2603/ActivePerl-5.26.3.2603-MSWin32-x64-a95bce075.exe
```

8、设置PATH 变量保证cl.exe、 rc.exe、perl.exe、nasm.exe都在PATH下

```
PATH=`cygpath -u "${VCINSTALLDIR}"`/bin:"$PATH"
PATH=`cygpath -u "${UniversalCRTSdkDir}"`../8.1/bin/x86:"$PATH"
PATH=/d/nasm-2.14.02:/d/Perl64/bin:$PATH
```

路径根据安装路径和操作系统可能不同，第一行设置cl.exe路径，第二行设置rc.exe，第三行设置perl.exe和nasm.exe

使用which命令查看是否正确

    which cl.exe 
    which rc.exe
    which perl.exe
    which nasm.exe

9、cd到nginx源码目录



10、执行configure命令，参数可以查看官方发布的nginx.exe -V查看，然后修改

    --with-openssl=openssl
    --with-pcre=pcre
    --with-zlib=zlib

根据需求增删一些module，比如我这里加上了--with-stream_ssl_preread_module，删除了一些不用的模块。


```
auto/configure \
--with-cc=cl \
--builddir=objs \
--with-debug \
--prefix= \
--conf-path=conf/nginx.conf \
--pid-path=logs/nginx.pid \
--http-log-path=logs/access.log \
--error-log-path=logs/error.log \
--sbin-path=nginx.exe \
--http-client-body-temp-path=temp/client_body_temp \
--http-proxy-temp-path=temp/proxy_temp \
--http-fastcgi-temp-path=temp/fastcgi_temp \
--http-scgi-temp-path=temp/scgi_temp \
--http-uwsgi-temp-path=temp/uwsgi_temp \
--with-cc-opt=-DFD_SETSIZE=1024 \
--with-pcre=pcre \
--with-zlib=zlib \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_stub_status_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_auth_request_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-stream \
--with-openssl=openssl \
--with-http_ssl_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module
```


11、执行nmake，如果有报错，需要具体分析，有可能某些module依赖的组件缺失，可以删减一些module尝试几次

    nmake -f objs/Makefile

12、编译成功后在objs中生成nginx.exe



13、将nginx.exe覆盖官方发布版中的nginx.exe，然后尝试运行。如果运行提示配置文件路径报错一般是--prefix= \参数的问题，这个参数需要设置为空

参考文档：

https://blog.csdn.net/travel981cn/article/details/82937834

https://blog.csdn.net/zhangge3663/article/details/80322770