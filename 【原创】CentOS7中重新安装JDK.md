#【原创】CentOS7中重新安装JDK

##1. 查找原有JDK
CentOS 7中默认安装了OpenJDK（与Oracle JDK的区别自行搜索），很多情况下我们需要重新安装。

检查原有版本JDK：
	
	# java -version
	
>java version "1.7.0_51"<br>
OpenJDK Runtime Environment (IcedTea6 1.11.1) (rhel-2.4.5.5.el7-x86_64)<br>
OpenJDK 64-Bit Server VM (build 24.51-b03, mixed mode)

查找JDK包文件：

	# rpm -qa | grep java
	
>java-1.7.0-openjdk-headless-1.7.0.51-2.4.5.5.el7.x86_64<br>
tzdata-java-2014b-1.el7.noarch<br>
python-javapackages-3.4.1-5.el7.noarch<br>
java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64<br>
javapackages-tools-3.4.1-5.el7.noarch<br>

##2. 卸载原有JDK
根据上面查找出来的包，依次卸载openjdk的包：

	# rpm -e --nodeps java-1.7.0-openjdk-headless-1.7.0.51-2.4.5.5.el7.x86_64
	# rpm -e --nodeps tzdata-java-2014b-1.el7.noarch
	# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.51-2.4.5.5.el7.x86_64

##3. 安装新版本JDK
从[Oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载最新版本JDK。

我下载的是jdk-8u66-linux-x64.gz文件，解压在我自定义的目录/usr/server中。

	# cd /usr/server/jdk1.8.0_66 
 	# bin/java

如果能打印出东西来，表明文件是正常的，可以继续进来；

##4. 配置环境变量
环境变量添加在 /etc/profile文件中：

	export JAVA_HOME=/usr/server/jdk1.8.0_66
	export JRE_HOME=/usr/server/jdk1.8.0_66/jre
	export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin	
然后使用命令使得修改生效：
	
	# source /etc/profile

##5. 测试

查看PATH中是否添加了JDK的目录：
	
	# echo $PATH
确认JDK版本：

	# java -version

>java version "1.8.0_66"<br>
Java(TM) SE Runtime Environment (build 1.8.0_66-b17)<br>
Java HotSpot(TM) 64-Bit Server VM (build 25.66-b17, mixed mode)<br>


