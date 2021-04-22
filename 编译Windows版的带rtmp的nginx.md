
# 编译Windows版的带rtmp的nginx #

看了一下这两年博客文章产量骤减，这样是不行的呀。

上次写关于nginx-rtmp-module的话题还是两年半之前的2016年。当年，即使是在Ubuntu上，安装带rtmp模块的nginx还要手动编译，因此有了那一篇文章。

不过Ubuntu 18.04开始，rtmp-module已经进了apt库，借着这两年直播越来越火的东风，从一个小众应用变成了众多用户使用的重要组件。同样地，有很多在生产环境上的大型应用也在使用这个nginx-rtmp-module。

如今，如果你是Ubuntu 18.04或者更新的系统，只需要运行下面的命令就行了

    apt install nginx libnginx-mod-rtmp

不过最近接到了一个需求，是一个老系统的监控平台改造。服务器是Winserver的，换成Ubuntu不太现实，而且平台也是ASP.NET写的，并且是用SQL Server做数据库的，整个就是微软全家桶的东西。

但是rtmp还要上，那怎么办呢？

那就编译windows版本呗。

于是我在踩了许多许多坑后决定写一篇文章。这篇文章不仅是记录了我这几天踩过一堆坑后，最终是如何把这个东西编译出来的。

更要感谢XhmikosR's build里找到了一个我一直搞不定的MSYS prebuilt。

以下开始正文：

## 1. 准备编译环境 ##

- 基础编译环境：VS2019，安装的时候勾上C++桌面开发，这样才会有我们需要的编译C的工具。
- Perl编译器：我用的是ActivePerl 5.26.3。这个在activestate官网上下载最新的win64版本安装就行了。
- MSYS：这就是把我搞死的地方的，踩的绝大多数的坑都是这里的。

下面直接放最终解决方案：去 http://xhmikosr.1f0.de/tools/msys/ 下一个MSYS_MinGW-w64_GCC_710_x86-x64_Full.7z 解压以后把bin目录加到path里，重启，就好了。

按正常方法来的话，应该是下载MinGW安装器，然后勾上MinGW-base和MSYS-base，点安装。等MinGW安装器给你把这两个东西装好。 然后同样的方法，把MSYS目录下的bin加到path里重启。但是由于大家都知道的某种原因，这种方式在我这里几乎不可用。

## 2. 准备源码和依赖库 ##

虽然官方编译指南建议使用hg，不过我还是用的惯git，反正github上的nginx镜像更新也挺快。就直接git clone https://github.com/arut/nginx-rtmp-module.git了。不过我推荐点进github的release，下载一个最新版本release的源码tar.gz包

然后随便放到哪解压。

解压之后的源码目录里面，创建一个叫objs的目录，然后再在里面创建一个lib目录，然后我们开始下依赖。

编译nginx本体需要的zlib、pcre和openssl，都直接用官网下载最新版本的源码tar.gz包即可。如果担心新版本编译可能会出问题，那就先参考官方编译指南中，这三个依赖库的版本，下载和那个一致的版本肯定是没问题的。

这些东西全部解压在lib目录下。而且注意不要有嵌套目录，就是lib/zlib打开就是源码，其他两个库同理。

然后再下载nginx-rtmp-module。可以直接git clone到lib目录下，也可以去release下载一个tar.gz包。

注意：下载的这些依赖库和nginx-rtmp-module的tar.gz包不要直接存在objs/lib目录里，也不要解压了就删掉，预防编译失败。

## 3. 编译准备 ##

我们需要改一点东西来完成编译。这又是一个坑，在编译过程中出现下列错误，编译失败：

    ......
    build/lib/nginx-rtmp-module/ngx_rtmp_core_module.c(611): error C2220: 警告被视为错误 - 没有生成“object”文件
    ......

所以我们把编译参数的W4改成W3。在源码的auto/cc/msvc的83行，修改为：

    CFLAGS="$CFLAGS -W3"

这样该警告就不会视为错误了。

在源码根目录启动MSYS的bash（如果你正确的加入了path又没有装WSL之类的重名工具，那么直接在cmd里敲bash回车就行了。）

然后参考官方编译指南的配置命令，注意修改几个依赖库的路径，正确指向objs/lib下的正确目录名的地方（很可能有版本号的差异）。

并在最后加上--add-module=objs/lib/nginx-rtmp-module

修改后的配置命令示例如下：

```
auto/configure \
    --with-cc=cl \
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
    --with-pcre=objs/lib/pcre-8.43 \
    --with-zlib=objs/lib/zlib-1.2.11 \
    --with-openssl=objs/lib/openssl-1.1.1b \
    --with-openssl-opt=no-asm \
    --with-http_ssl_module \
    --add-module=objs/lib/nginx-rtmp-module
```

按回车执行。如果没有报错，现在就生成了一份对应的Makefile。

## 4. 编译 ##

在开始菜单的VS2019目录下可以找到“Developer Command Prompt for VS 2019”，启动它，并转到nginx源码目录下。

键入命令

    nmake

开始编译

如果编译失败了，可以使用nmake clean清理源码目录。但是注意objs目录会被整个删掉，因此你需要重新创建objs/lib并重新往里放置依赖库。

## 5. 编译成功之后的事 ##

编译完成了之后，在objs目录下就会生成nginx.exe了。由于windows下的nmake install很混乱。因此这里我们手工操作。

在随便哪个空目录把nginx.exe复制过去，这个就将作为nginx的主目录。

在和nginx.exe同级别，创建conf、html、logs、temp四个目录。然后把源码目录conf里的所有文件复制到conf目录下面。把源码目录docs/html里的所有文件复制到html目录下。

创建一个bat文件用于关闭nginx，内容如下：

    @echo off
    nginx -s stop

启动nginx时，只需双击nginx.exe即可。你将看到黑窗口一闪而过，这就是启动了。需要关闭的时候请双击刚才创建的那个bat。

## 6. 开启rtmp功能 ##

编译conf/nginx.conf，按照nginx-rtmp-module的官方教程编写rtmp配置，然后重启nginx就行了。

注意，在Windows下：execs、static pulls和auto_push这几个功能无法使用。