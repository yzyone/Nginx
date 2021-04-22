
# windows编译64位nginx 注意事项 #

nginx官方给出了在windows下编译的介绍

    http://nginx.org/en/docs/howto_build_on_win32.html

不过如果严格按照他的介绍来做，会出现很多问题，所以这里记录一些修正，以免忘记

参考链接

    https://blog.csdn.net/u010505059/article/details/92661913

里面介绍了比较详细的步骤，这里还要再做一点补充，以便使其更完整

**第1步是下载nginx的源代码**

nginx官方给出的.tar.gz包缺文件

    http://nginx.org/en/download.html

如果下载这个压缩包，在编译时会出现

    NMAKE : fatal error U1073: don't know how to make 'src/os/win32/ngx_win32_config.h'

 所以到

    https://github.com/nginx/nginx/releases

下载zip包。参考链接中的文章，直接使用了hg，所以没有遇到这个问题，我这里git和hg都很慢，所以一开始用的是官方源码包，就出了问题。原因在于官方压缩包丢失了src/os目录。

**第2步是准备依赖包**

首先当然是MSVC编译器，笔者用的是MSVC2017社区版，也就是VC14.1或者叫VC15（因为MSVC2019版本号直接跳到VC16了）

nginx需要使用3个额外的软件包openssl、pcre、zlib，在编译nginx之前，会编译这3个包

其中openssl配置时需要调用perl，而且需要5.30以上版本，我在这里栽了跟头。因为我的电脑上装了git for windows，而它自带的perl是5.26版的，所以出了问题。perl在很多软件中都有包含，比如msys2、texlive等，在使用之前务必使用

    perl -v

检查一下，看看perl版本对不对，如果系统里有多个perl，则需要把5.30或更新版放到path的最前面，以便识别出来

另外如果openssl的版本是1.0.x系列的，那么涉及到汇编代码时，MSVC会用ml或者ml64来替代；而如果openssl是1.1.x系列的，在涉及到汇编代码时，会调用nasm，此时如果系统path找不到nasm，则会出错。之前别的博文有讲nginx不能搭配openssl-1.1.x系列，就是这个原因；其实是可以的，毕竟官方的编译指南里都是用的1.1.x系列。

将这3个软件解压缩到nginx源码目录下的objs/lib目录备用。

如果openssl的版本是1.0.x系列的，还可以考虑vcpkg自带的补丁，在vcpkg中如果安装过openssl，它目前使用的openssl版本是1.0.1c，自带了5个补丁，其中有两个补丁是分别针对32位和64位的，所以实际上是4个补丁，根据用户的需要，可以选择一个打上。这几个补丁在`vcpkg\ports\openssl-windows`

    patch -p1 < filename.patch

如果使用1.1.x系列的openssl版本，则暂时还没有类似补丁

    第3步是给nginx打补丁

由于Nignx没有提供相关配置项改变缺省banner，所以我们需要改变源码，然后重编译和重新安装一下,具体操作：

找到/nginx/src/http/ngx_http_header_filter_module.c文件（我的nginx的安装目录为/nginx），修改以下变量的声明：

```
static u_char ngx_http_server_string[] = "Server: nginx" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;  
```
    
修改为

```
static u_char ngx_http_server_string[] = "Server: " CRLF;
static u_char ngx_http_server_full_string[] = "Server:  " CRLF;
static u_char ngx_http_server_build_string[] = "Server: " CRLF;
```

官方的windows版nginx是针对32位的，如果要用64位的，需要编辑一下代码

打开`nginx\auto\lib\openssl\makefile.msvc`文件

找到“VC-WIN32”替换为“VC-WIN64A”；

“if exist ms\do_ms.bat”替换为“if exist ms\do_win64a.bat”；
“ms\do_ms”替换为“ms\do_win64a”。

**第4步是配置nginx编译选项**

配置nginx要使用msys调用nginx的配置脚本。msys2的官网是

    http://www.msys2.org/

不过太慢，所以我一般都是用国内镜像，清华或者中科大或者华为云都有

    http://mirrors.ustc.edu.cn/msys2/

    https://mirrors.tuna.tsinghua.edu.cn/msys2/

    https://mirrors.huaweicloud.com/msys2/

下载解压以后，直接运行根目录的msys2.exe或者mingw64.exe都可以。msys2会把windows的盘映射为/c、/d、/e、/f……，用cd命令进入nginx源代码目录后

```
./auto/configure --with-cc=cl --builddir=objs --prefix= \
--conf-path=conf/nginx.conf --pid-path=logs/nginx.pid \
--http-log-path=logs/access.log --error-log-path=logs/error.log \
--sbin-path=nginx.exe --http-client-body-temp-path=temp/client_body_temp \
--http-proxy-temp-path=temp/proxy_temp \
--http-fastcgi-temp-path=temp/fastcgi_temp \
--http-scgi-temp-path=temp/scgi_temp \
--http-uwsgi-temp-path=temp/uwsgi_temp \
--with-cc-opt=-DFD_SETSIZE=1024 --with-pcre=objs/lib/pcre-8.41 \
--with-zlib=objs/lib/zlib-1.2.11 --with-openssl=objs/lib/openssl-OpenSSL_1_1_1c \
--with-select_module --with-http_ssl_module
```

这里在每行的末尾都添加了换行符，这是为了提醒msys2这一行还没结束，不过千万不要试图在复制之后直接粘贴到msys2控制台，换行符识别不出来，执行会出错。读者需要把这段命令复制到文本编辑器，然后在编辑器中编辑后再粘贴到msys2控制台。注意我用的pcre/openssl/zlib库的版本号，如果你的不一样，那要自行修改。

成功之后会在objs目录下生成很多文件。

这一步还不需要perl和nasm，也就是说这两个命令行工具的配置可以等到下一步之前做。

(这里会发现有一个异样的提示，auto/cc/msvc: line 117: [: : integer expression expected ，只要修改文件 $nginx源码目录\auto\cc\msvc 即可，在 echo " + cl version: $NGX_MSVC_VER" 的前面加入一行 NGX_MSVC_VER=15.00 ，当然不修改也不会影响后续的编译。 )

**第5步是编译**

首先把perl和nasm加入系统path，注意识别到的perl必须是5.30或更高版本

用MSVC的控制台进入nginx的源码目录，然后检查一下

    perl -v

    nasm -v

如果没有问题，那么

    nmake -f objs/Makefile

nmake会自动处理openssl、pcre和zlib，然后自动编译nginx，最后会在objs目录下生成nginx.exe文件，那3个外部库会被静态链接到nginx.exe中，在运行时不需要额外附加dll文件

如果你的系统path里没有sed，那么编译的最后会提示找不到sed，不过我的系统里有git for windows，所以有sed，最后没有任何出错提示。不管这里有没有出错提示，nginx.exe已经生成，可以用了。

如果你对系统有洁癖，那么在这之后就可以把perl和nasm相关的配置从系统中删掉了

**第6步是打包**

从官方下载32位版的nginx包，然后用刚刚编译好的64位的nginx.exe替换就可以了，其他的目录下都是文本配置文件，最好是用官方包自带的，不要自己创建。

————————————————

版权声明：本文为CSDN博主「silent_missile」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/silent_missile/article/details/99317811