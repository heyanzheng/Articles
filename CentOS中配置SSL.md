#【原创】CentOS中配置SSL（翻译）
官方文章：https://wiki.centos.org/HowTos/Https<br>
（根据自己的理解而翻译，内容有所增减）

这篇文章将指导你如何配置HTTPS；教程中将使用自签名证书来作为示例。

##1. 获取需要的软件
根据需要来安装OpenSSL和mod_ssl模块。

>CentOS 7 中默认已安装了OpenSSL，使用rpm -q openssl 即可查看系统中使用的版本

如果没有安装，使用如下命令同时安装OpenSSL和mod_ssl：

	# yum install mod_ssl openssl
>推荐使用yum而不是rpm，yum会自动下载各自的依赖包

##2. 获取自签名证书
使用OpenSSL即可获得自签名证书。如果你现在正在给企业级服务器配置，建议你从信任的证书发放机构中获取；如果你和我一样只是在私人机器上测试，自签名证书就够了。获取之前，最好使用root权限；不然命令中会有一大堆的sudo...

> Generate private key 

	# openssl genrsa -out ca.key 2048
> Generate CSR

	# openssl req -new -key ca.key -out ca.csr
> Generate Self Signed Key

	# openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
> Copy the files to the correct locations

	# cp ca.crt /etc/pki/tls/certs
	# cp ca.key /etc/pki/tls/private/ca.key
	# cp ca.csr /etc/pki/tls/private/ca.csr
> Tips: 如果使用的是SELinux版本的服务器，确保使用的是拷贝命令（cp）而不是移动命令（mv）。

如果你已经移动了文件，那可以使用如下命令来纠正下：

	# restorecon -RvF /etc/pki
然后更新下SSL的配置文件

	# vi +/SSLCertificateFile /etc/httpd/conf.d/ssl.conf
更改证书的文件位置

	# SSLCertificateFile /etc/pki/tls/certs/ca.crt
更改证书key的文件位置

	# SSLCertificateKeyFile /etc/pki/tls/private/ca.key
保存退出（:wq）后重启Apache

	# /etc/init.d/httpd restart   ( centos7后可使用systemctl restart httpd )
做完这些之后，你应该就可以使用https来访问网站了；浏览器会提醒你接受证书。

##3. 设置虚拟主机
443的配置和80的配置一样，典型的80端口配置如下：

	<VirtualHost *:80>
        <Directory /var/www/vhosts/yoursite.com/httpdocs>
        AllowOverride All
        </Directory>
        DocumentRoot /var/www/vhosts/yoursite.com/httpdocs
        ServerName yoursite.com
	</VirtualHost>

在443端口增加一个网站的话，你需要在以上配置顶部增加一条：

	NameVirtualHost *:443

所以配置文件应该是这样的：

	<VirtualHost *:443>
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/ca.crt
        SSLCertificateKeyFile /etc/pki/tls/private/ca.key
        
        <Directory /var/www/vhosts/yoursite.com/httpsdocs>
        	AllowOverride All
        </Directory>
        
        DocumentRoot /var/www/vhosts/yoursite.com/httpsdocs
        ServerName yoursite.com
	</VirtualHost>
重新启动：

	# /etc/init.d/httpd restart   ( centos7后可使用systemctl restart httpd )

##4. 配置防火墙
此时如果使用https访问不了，那你需要看下防火墙中是否开启了443端口：

	iptables -A INPUT -p tcp --dport 443 -j ACCEPT
如果没有，添加上后保存配置：	
	
	# /sbin/service iptables save
	# iptables -L -v
