#【原创】CentOS7中源码编译安装Nginx

##1 下载源码
官方地址：http://nginx.org/en/download.html
当前稳定版最新版本为1.8.0，所以此次也以1.8.0为演示说明。
下载完后为nginx-1.8.0.tar.gz。

##2 解压
首先使用FTP工具将下载好的文件上传到服务器中，我使用的是FileZilla，目录为~/tmp；<br>
然后使用命令解压：

	# tar -xzf nginx-1.8.0.tar.gz

##3 安装编译环境
我当前使用的是CentOS 7 Minimal-1503-01环境，所以有些已经安装，但也有许多没有安装，这些在下面会一一说明。
因为Nginx完全采用C\C++语言来编写，所以要编译的话，首先需要GCC\G++之类的开发库要提前安装好。所以，我们可以使用如下命令来安装：

	# yum -y install gcc automake autoconf libtool make
	# yum install gcc gcc-c++

若没有更改yum默认仓库地址的话，安装可能需要等待一会。

另外，编译还需要pcre、zlib和ssl，pcre是为了实现重写rewrite，zlib是为了gzip压缩。

我的CentOS7中已安装了pcre，可以不用安装了；如果没有安装，可利用wget下载源码包make install安装，注意的是此时需要g++编译器。

	# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.36.tar.gz
	# ./configure
	# make
	# make install

zlib也是一样，下载源码后configure后make install安装即可。

CentOS7中默认安装了OpenSSL，可以不用安装；如果没有安装，wget下来后解压即可。

##4 Configure

	./configure --prefix=/usr/server/nginx \
	--sbin-path=/usr/server/nginx/sbin/ \
	--conf-path=/usr/server/nginx/nginx.conf \
	--pid-path=/var/run/nginx/nginx.pid \
	--lock-path=/var/lock/nginx/nginx.lock \
	--error-log-path=/var/log/nginx/error.log \
	--http-log-path=/var/log/nginx/access.log \
	--user=nginx \
	--group=nginx \
	--with-http_ssl_module \
	--with-http_gzip_static_module \
	--with-http_stub_status_module \
	--with-http_realip_module \
	--with-pcre=/usr/local/src/pcre-8.34 \
	--with-zlib=/usr/local/src/zlib-1.2.8 \
	--with-openssl=/usr/local/src/openssl-1.0.1c

>--prefix     #nginx安装目录，默认在/usr/local/nginx<br>
>--sbin-path     #设置nginx的可执行文件的路径，默认为prefix/sbin/nginx<br>
>--conf-path      #设置nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/conf/nginx.conf<br>
>--error-log-path     #设置主错误，警告，和诊断文件的名称。安装完成后，可随时改变的文件名 ，在nginx.conf配置文件中使用的error_log指令。默认情况下，文件名 为prefix/logs/error.log<br>
>--http-log-path=path      #设置主请求的HTTP服务器的日志文件的名称。安装完成后，可随时改变的文件名 ，在nginx.conf配置文件中使用的access_log指令。默认情况下，文件名为prefix/logs/access.log<br>
>--pid-path    #pid问件位置，默认在logs目录<br>
>--lock-path    #lock问件位置，默认在logs目录<br>
>--user=name      #设置nginx工作进程的用户。安装完成后，可随时更改在nginx.conf配置文件中使用的 user指令。默认的用户名是nobody<br>
>--group=name      #设置nginx工作进程的用户组。安装完成后，可以随时更改的名称在nginx.conf配置文件中使用的指令。默认的为非特权用户<br>
>--with-select_module  --without-select_module      #启用或禁用构建一个模块来允许服务器使用select()方法。该模块将自动建立，如果平台不支持的kqueue，epoll，rtsig或/dev/poll<br>
>--with-poll_module  --without-poll_module      #启用或禁用构建一个模块来允许服务器使用poll()方法。该模块将自动建立，如果平台不支持的kåqueue，epoll，rtsig或/dev/poll<br>
>--with-http_ssl_module     #使用https协议模块。默认情况下，该模块没有被构建。前提是openssl与openssl-devel已安装<br>
>--with-http_stub_status_module     #用来监控 Nginx 的当前状态<br>
>--with-http_realip_module     #通过这个模块允许我们改变客户端请求头中客户端IP地址值(例如X-Real-IP 或 X-Forwarded-For)，意义在于能够使得后台服务器记录原始客户端的IP地址<br>
>--with-mail   #启用 IMAP4/POP3/SMTP 代理模块<br>
>--with-mail_ssl_module   #启用 ngx_mail_ssl_module<br>
>--add-module      #添加第三方外部模块，如nginx-sticky-module-ng或缓存模块。每次添加新的模块都要重新编译<br>
>--without-http_gzip_module      #删除压缩的HTTP服务器的响应模块。编译并运行此模块需要zlib库<br>
>--without-http_rewrite_module     #删除重写模块。编译并运行此模块需要PCRE库支持<br>
>--without-http_proxy_module      #删除http_proxy模块<br>

	# make
	# make install


