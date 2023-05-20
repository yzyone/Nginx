# ubuntu下安装nginx时依赖库zlib，pcre，openssl安装方法

转自：https://blog.csdn.net/z920954494/article/details/52132125

错误：./configure: error: the HTTP rewrite module requires the PCRE library. You can either disable the module by using --without-http_rewrite_module option, or install the PCRE library into the system, or build the PCRE library statically from the source with 

最近刚开始学习[Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020)，安装Nginx时需要先安装依赖包，Ubuntu中不像centOS等使用yum直接在线安装，在网上找了好多方法，最后发现以下方法好用：

首先使用dpkg命令查看自己需要的软件是否安装。

例如查看[zlib](https://so.csdn.net/so/search?q=zlib&spm=1001.2101.3001.7020)是否安装：

`dpkg -l | grep zlib`

解决依赖包[openssl](https://so.csdn.net/so/search?q=openssl&spm=1001.2101.3001.7020)安装，命令：

`sudo apt-get install openssl libssl-dev`

解决依赖包pcre安装，命令：

`sudo apt-get install libpcre3 libpcre3-dev`

解决依赖包zlib安装，命令：

`sudo apt-get install zlib1g-dev`