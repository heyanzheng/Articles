
>好像从CentOS 7开始，系统默认安装的是Mariadb数据库，而不再是MySQL。至于为什么，问问Oracle吧~~<br>

	$ rpm -qa | grep maria
	mariadb-5.5.44-2.el7.centos.x86_64
	mariadb-libs-5.5.44-2.el7.centos.x86_64
	mariadb-devel-5.5.44-2.el7.centos.x86_64

从上面可以看出，系统中安装了部分mariadb的组件，但并不完全；下面我们使用yum来完整安装；

	$ yum -y install mariadb*

启动mariadb数据库服务

	$ systemctl start mariadb

测试能否连通

	$ mysql
	Welcome to the MariaDB monitor.  Commands end with ; or \g.
	Your MariaDB connection id is 2
	Server version: 5.5.44-MariaDB MariaDB Server
	...

从中可以看出，连接成功！

exit退出后，开始安全设置向导

	$ mysql_secure_installation
	
	NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

	In order to log into MariaDB to secure it, we'll need the current
	password for the root user.  If you've just installed MariaDB...

其中会提示为root设置密码，删除匿名帐号，设置是否取消root远程登录，设置是否删除test库和对test库的访问权限，刷新授权表使修改生效

进入MySQL的命令

	$ mysql -u root -p
	Enter password:

自此，Mariadb设置完成；最后设置数据库服务自动开启：

	$ systemctl enable mariadb

***

安装完后，Mariadb默认的数据文件目录为/var/lib/mysql，若想改变这个目录的话（生产环境下多为数据和程序分离或磁盘原因）

首先停掉数据库服务

	$ systemctl stop mariadb

将整个/var/lib/mysql文件夹复制到目标文件夹（如/usr/service）

	$ cp -a /var/lib/mysql/* /usr/service/

> -a 将文件属性一起拷贝，否则可能出现很多问题

编辑Mariadb的配置文件/etc/my.cnf

	$ cp my.cnf my.cnf.ogi
	$ vi my.cnf

修改`datadir=/usr/service`，不需要修改socket等其它信息

可顺便看下my.cnf.d/server.cnf client.cnf 文件信息，看是否有datadir信息；若没有，则不再需要修改

	$ chown -R mysql:mysql /usr/service

启动服务

	$ systemctl start mariadb


