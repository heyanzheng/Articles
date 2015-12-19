
>系统为CentOS 7-1503 最小安装包，FTP工具为xftp 5

##安装

查看是否有vsftpd服务：`$ chkconfig --list`

是否安装过vsftpd包：`rpm -qa | grep vsftpd`

如果都没有，开始安装：`yum -y install vsftpd`

1. 查看配置文件所在位置：`rpm -qc vsftpd`
2. 备份配置文件

	$ cd /etc/vsftpd
	$ cp vsftpd.conf vsftpd.conf.ogi

##创建登录用户

	$ vi vsftpuser
	$ hyz
	$ ssaa

:wq保存退出后，根据这个文件再创建DB文件

	$ db_load -T -t hash -f vsftpuser vsftpuser.db

如果db_load命令找不到，则需要安装db4*包 <br>
> yum install db4 db4-utils

查看后可知属于Berkeley DB;

	$ file vsftpuser.db
	$ vsftpuser.db: Berkeley DB (Hash, version 9, native byte-order)


##修改配置文件

修改认证中的设置

	$ cd /etc/pam.d
	$ vi vsftpd

将文件中所有auth和account开头的配置行均注释，并在最后添加如下内容：

	auth required pam_userdb.so db=/etc/vsftpuser
	account required pam_userdb.so db=/etc/vsftpuser

> 指向的就是生成的DB文件

打开/etc/vsftpd/vsftpd.conf文件，将`anonymous_enable=YES`修改`anonymous_enable=NO`，并在文件最后添加：

	virtual_user_local_privs=YES
	chroot_local_user=YES
	allow_writeable_chroot=YES

##配置防火墙等

修改防火墙设置：

	$ firewall-cmd --permanent --zone=public --add-service=ftp
	$ success

>CentOS 7中默认防火墙为firewalld而不再是iptables

返回success表示设置成功；

	$ firewall-cmd --reload

记得防火墙需要重启；

修改SELinux bool值

	$ getsebool -a | grep ftp
	$ setsebool ftp_home_dir on
	$ setsebool ftpd_full_access on

>此种写法重启后将无效，若 set -P ftpd_full_access on，则将配置写入磁盘，重启后配置仍在，但耗时较长。


##服务设置

将vsftpd设置为开机启动过

	$ systemctl enable vsftpd

重启服务

	$ systemctl restart vsftpd

查看服务状态

	$ systemctl status vsftpd

若状态为 `active (running)` 则表示服务正在运行，可以使用ftp工具链接测试；

##连接测试

使用ftp工具或直接在浏览器中输入ftp://ip地址，用户名和密码即在文件中设置的数据（hyz，ssaa）登录，连接成功！

vsftpd默认访问的地址为/home/xxxx目录下，xxxx即当前用户；