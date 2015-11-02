#【原创】Linux 常用命令记录
##RPM
	$ wget http://xx.org/xxx.x86_64.rpm
	$ rpm -ivh xxx.rpm
安装显示安装进度 --install--verbose--hash
	
	$ rpm -q httpd 查看是否安装
	$ rpm -ivh --test xx-1.fc4.i386.rpm 检查依赖关系，并非安装
	$ rpm -qa | grep httpd    搜索httpd是否安装
	$ rpm -qi httpd   httpd的信息
	$ rpm -ql httpd   列出文件安装目录
	$ rpm -qpR xxx.rpm  查看包的依赖关系
	$ rpm -e --nodeps 不检查包的依赖而删除

##YUM
可解决增加或删除rpm包时遇到的倚赖性问题；推荐使用YUM而不是RPM；<br>
>查看系统默认安装的yum

	$ rpm -qa|grep yum

yum的配置文件
>main 部分定义了全局配置选项，整个yum 配置文件应该只有一个main。常位于/etc/yum.conf 中<br>
>repository 部分定义了每个源/服务器的具体配置，可以有一到多个。常位于/etc/yum.repo.d 目录下的各文件中
	
	$ cat /etc/yum.conf
	$ cat /etc/yum.repos.d/CentOS-Base.repo

命令：
> -y 当安装过程提示选择全部为"yes"<br>
> -q 不显示安装的过程<br>

	$ yum install httpd
	$ yum update httpd
	$ yum list
	$ yum info httpd
	$ yum remove httpd
	$ yum deplist httpd (httpd的依赖包情况)
	$ yum clean packages (清除缓存目录下的软件包)

##文件和文件夹
###复制  cp

	$ cp xxx.tar.gz ~/usr
>-a:是指archive的意思，也说是指复制所有的目录<br>
> -r: 递归复制，用于目录的复制操作<br>
> -p:与文件的属性一起复制，而非使用默认属性<br>
> -s:复制成符号连接文件(symbolic link)，即“快捷方式”文件<br>

###移动  mv

	$ mv xxx.tar.gz ~/usr
>-f:force，强制直接移动而不询问<br>
>-u:若目标文件已经存在，且源文件比较新，才会更新

	$mv xxx.tar.gz  yyy.tar.gz
>修改文件名

###删除  rm

	$ rm -f 目录或文件
> -f:强制删除<br>
> -r:递归删除，用于目录删除


##More/Cat
more命令是将文件以一页一页的显示，方便使用者逐页阅读；最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示；<br>
命令参数：
>-n       定义屏幕大小为n行<br>
>+n      从笫n行开始显示<br>
>-c       从顶部清屏，然后显示<br>
>-s       把连续的多个空行显示为一行<br>

操作命令
>Enter    向下n行，需要定义。默认为1行<br>
>空格键  向下滚动一屏<br>
>Ctrl+B  返回上一屏<br>
>=       输出当前行的行号<br>
>V      调用vi编辑器<br>
>!       调用Shell，并执行命令<br>
>q       退出

	$ ls -l | more 5
>列出目录下的文件，每页显示5个文件信息

<br>

cat(concatenate files and print on the standard output)命令主要用来查看文件内容，创建文件，文件合并，追加文件内容等功能；
>-s  当遇到有连续两行以上的空白行，就代换为一行的空白行<br>
>-n  由 1 开始对所有输出的行数编号<br>
>-b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号<br>
>-E, --show-ends 在每行结束处显示 $

	$ cat file1.txt file2.txt > file3.txt  (如果file3.txt文件以前有内容，则先会清除它们然后再写入合并后的内容)
	$ cat file1.txt file2.txt >> file3.txt  (不会清除file3.txt里面原有内容)

##MD5
	$ md5sum xxx.tar.gz > a.txt
	$ cat a.txt

##打包和压缩
Linux中的很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，需要先借助另外的工具将这一大堆文件先打成一个包，然后再用压缩程序进行压缩。

	$ tar -cf all.tar *.jpg

这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。

	$ tar -rf all.tar *.gif

这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。

	$tar -uf all.tar logo.gif

这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。

	$tar -tf all.tar

这条命令是列出all.tar包中所有文件，-t是列出文件的意思

> tar还可以在打包或解包的同时调用其它的压缩程序，比如调用gzip、bzip2等。

###tar调用gzip
gzip是GNU组织开发的一个压缩程序，.gz结尾，与gzip相对的解压程序是gunzip；tar中使用-z这个参数来调用gzip。

	$ tar -czf all.tar.gz *.jpg

这条命令是将所有.jpg的文件打成一个tar包，并且将其用gzip压缩，生成一个gzip压缩过的包，包名为all.tar.gz

	$ tar -xzf all.tar.gz

这条命令是将上面产生的包解开。

###tar调用bzip2
bzip2是一个压缩能力更强的压缩程序，.bz2结尾；与bzip2相对的解压程序是bunzip2。tar中使用-j这个参数来调用bzip2。

	$ tar -cjf all.tar.bz2 *.jpg

这条命令是将所有.jpg的文件打成一个tar包，并且调用bzip2压缩，生成一个bzip2压缩过的包，包名为all.tar.bz2

	$ tar -xjf all.tar.bz2

这条命令是将上面产生的包解开。
###tar调用compress 
compress也是一个压缩程序，但使用compress的人不如gzip和bzip2的人多，.Z结尾；与compress相对的解压程序是uncompress。tar中使用-Z这个参数来调用gzip。

	$ tar -cZf all.tar.Z *.jpg

这条命令是将所有.jpg的文件打成一个tar包，并且调用compress压缩，生成一个uncompress压缩过的包，包名为all.tar.Z

	$ tar -xZf all.tar.Z

这条命令是将上面产生的包解开
###zip压缩
把一个文件abc.txt和一个目录dir1压缩成为yasuo.zip：

	$ zip -r yasuo.zip abc.txt dir1
下载了一个yasuo.zip文件，想解压缩：

	$ unzip yasuo.zip
>-q：执行时不显示任何信息<br>
>-p：与-c参数类似，会将解压缩的结果显示到屏幕上，但不会执行任何的转换

当前目录下有abc1.zip，abc2.zip和abc3.zip，想一起解压缩它们：

	$ unzip abc\?.zip
>?表示一个字符，如果用*表示任意多个字符

压缩文件large.zip很大，不想解压缩，只想看看它里面有什么：
	
	$ unzip -v large.zip

下载了一个压缩文件large.zip，验证压缩文件是否完整：

	$ unzip -t large.zip

###7z解压
7zip命令有7z和7za；<br>7za是精简版，部分格式不支持；7z是全功能版的，建议使用7z

	$7za x xxx.7z
>也可以用于打包解压zip文件

##网络
虚拟机上不了网，ping\curl都无法解析host时，即有可能网卡被禁用了，可采取如下措施：<br>
1.查看系统网络配置

	$ip addr
2.修改网卡配置

	$vi /etc/sysconfig/network-scripts/ifcfg-xxx

3.选择使用共享IP的对应网卡文件，打开，修改ONBOOT为yes，启用此网卡<br>
4.保存退出，重启网络

	$service network restart
> 或者是  /etc/init.d/network restart

5.再次连接即可

***

列出所有端口：
	
	$ netstat -ntlp
查看80端口占用情况:
	
	$ lsof -i tcp:80

##系统服务

	$ systemctl enable httpd.service (在centos7中代替chkconfig httpd on) 设置为自动启动
	$ systemctl disable httpd.service 设置为不自动启动
	$ systemctl status httpd
	$ systemctl start httpd
	$ systemctl stop httpd
	$ systemctl restart httpd
	$ systemctl list-units --type=service  显示所有已启动的服务


##权限

r=4，w=2，x=1，三种身份的权限是r+w+x的和，如果没有相应的权限，则值为0

owner=rwx=4+2+1=7，group=rwx=4+2+1=7，others=---=0+0+0=0，所以这个文件的将改变权限值为770
	
	$ chmod 770 error.log
