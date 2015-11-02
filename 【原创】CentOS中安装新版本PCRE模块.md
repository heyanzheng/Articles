#【原创】CentOS中安装新版本PCRE模块

##1. 查询系统中原有版本
一般系统中都会默认安装一个版本，所以我们需要先找出原有版本：

	# rpm -q pcre

>pcre-8.32-12.el7.x86_64

系统中文件包的位置：
	
	# rpm -ql pcre
	
>/usr/lib64/libpcre.so.1<br>
/usr/lib64/libpcre.so.1.2.0<br>
/usr/lib64/libpcre16.so.0<br>
/usr/lib64/libpcre16.so.0.2.0<br>
/usr/lib64/libpcre32.so.0<br>
/usr/lib64/libpcre32.so.0.0.0<br>
/usr/lib64/libpcrecpp.so.0<br>
/usr/lib64/libpcrecpp.so.0.0.0<br>
/usr/lib64/libpcreposix.so.0<br>
/usr/lib64/libpcreposix.so.0.0.1<br>
/usr/share/doc/pcre-8.32<br>
/usr/share/doc/pcre-8.32/AUTHORS<br>
/usr/share/doc/pcre-8.32/COPYING<br>
/usr/share/doc/pcre-8.32/README<br>
...<br>

##2. 不要删除原有版本
网上有许多教程指导说，在删除系统自带的PCRE之前，要先备份一下libpcre.so.0等一些文件，但从我安装失败几次的经验后，我发现系统不能删除原有版本PCRE；一旦删除后，ls、cp等很多命令都失效了，也就能用个cd命令...基本等于废了，至少我用的CentOS 7是这样的。

##3. 自定义目录安装
将下载的最新版本PCRE上传到指定目录，我将其拷贝到/usr/local/pcre目录中。

	# tar -xzf pcre2-10.20.tar.gz
	# cd pcre2-10.20
	# ./configure
	# make
	# make install

完成！

以后需要PCRE模块，指定这个目录即可。

切记，CentOS 7千万别删除原有版本PCRE，不然你会后悔的！
