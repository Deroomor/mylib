# mylib
Linux笔记

## centos6
### 网络设置
#### 本机所有的网卡设备 /etc/udev/rules.d/70-persistent-net.rules  
#### SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:15:5d:38:67:03", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"  

#### 各个网卡的配置信息 /etc/sysconfig/network-scripts/ifcfg-eth0  ifcfg-[网卡设备名称，eth0,eth1] 对应网卡设备信息里面的NAME
#### DEVICE=eth0 //对于网卡设备的NAME
#### HWADDR=00:15:5D:38:67:03    //对于网卡设备的ATTR{address}
#### TYPE=Ethernet
#### UUID=bbac13e3-9e0a-449d-aa13-e5e5b072cac4
#### ONBOOT=yes  //这个默认是no,表示开机不启用该网卡
#### NM_CONTROLLED=yes
#### BOOTPROTO=dhcp

###	安装桌面 yum groupinstall Desktop    X Window System  Chinese Surport
### yum info mysql* available

### Docker
docker -v
### Docker镜像管理
docker pull 镜像名称  //获取镜像  
docker rmi 镜像ID   //删除镜像  
docker images   //列出所有的镜像  
### 容器管理
docker create 镜像名    //新增容器  
docker rm 容器ID    //删除容器  
docker run 参数  
--name  自定义容器名称  
-d  容器后台运行  
-p  当前系统端口：容器端口  端口映射（容器内部端口映射外部）  
-v  当前系统目录：容器目录   目录映射  
docker run -it --name 容器名称 repository:tag /bin/bash //以交互方式启动

docker exec -i -t 通过docker ps查看的name名 /bin/bash

开放TCP连接
修改/etc/docker/daemon.json，没有就新增
内容
  {"hosts":[
"unix:///var/run/docker.sock",
"tcp://0.0.0.0:2375"
]}
  重启服务或者服务器，查看2375端口是否开放，ss -l |grep -Po '\s[^\s]*2375\s'

客户端连接服务端：docker -H tcp://192.168.56.101:2375 version
docker -H 192.168.56.101:2375 version

docker load：导入镜像到本地
docker load [OPTIONS]
-i：从tar文件读取
-q：禁止读入输出
docker load -i mytomcat_v1.tar

客户端每次运行都需要指定 -H tcp://192.168.137.21:2375 ， 比较麻烦，
可以 通过：   export DOCKER_HOST="tcp://192.168.137.21:2375"  建立一个环境变量，后面就可以直接运行：   docker info 也能得到远程docker 的信息了
windows环境下 cmd    使用 set DOCKER_HOST=tcp://192.168.56.101:2375

dotnet交叉编译 发布到linux64 dotnet 命令为：dotnet publish -r linux-x64 //不需要这样编译
dotnet publish  //用这个就可以，为当前目录中的项目创建一个 依赖于运行时的跨平台二进制文件：

##在netcore的项目文件中，让dockerfile始终复制到发布后的目录<None Include="Dockerfile" CopyToPublishDirectory="Always" />


发布以后，使用docker build发布以后的文件
Dockerfile内容
FROM microsoft/aspnetcore:2.0
#指定一个workdir，目录随便
WORKDIR /app
#把相对dockerfile所在目录的相对路径的文件 复制到 workdir相对路径的位置
COPY . .
#声明系统运行需要用到的端口，只是声明，方便创建容器的人知道需要处理哪些端口映射
expose 80
#等价于cmd的dotnet命令 
ENTRYPOINT ["dotnet", "Azeroth.Klz.dll"]

在windows环境下publish项目
在windows的docker客户端build一个镜像 
docker build -t 景象名称 dockerfile所在的路径
docker build -t wch .\bin\Debug\netcoreapp2.0\publish\

在windows的docker客户端run一个容器
docker run -it --rm -p 容器外部端口:容器内部端口 --name 容器名称 镜像名称
docker run -it --rm -p 5000:80 --name wch123 wch

访问容器所在的linux的地址的http://192.168.56.101:5000/api/values

#发布程序，build镜像，创建容器及运行，删掉tag是none的镜像（也就是老版本的镜像）做成一个cmd,内容为：
dotnet publish
docker build -t wch .\bin\Debug\netcoreapp2.0\publish\
docker stop wch123
docker rm wch123
docker run -d -it --rm -p 5000:80 --name wch123 wch
docker images|grep none|awk '{print $3 }'|xargs docker rmi

FAQ:
https://docs.microsoft.com/zh-cn/dotnet/core/tools/dotnet-publish

// 停止所有容器
docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker stop
// 删除所有停止的容器
docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker rm
// 删除所有tag标签是none的镜像
docker images|grep none|awk '{print $3 }'|xargs docker rmi

### 网络管理
默认本机和容器

## centos7
使用界面配置  nmtui  
查看ip信息  ip address  
### centos7 网卡命名规则：
### en： ethernet以太网卡
### o：主板集成网卡
### p：PCI独立网卡
### s：热插拔网卡
### nnn数字：MAC+主板信息（生产唯一序号）
### 如：ifcfg-ens33 （以太热插拔）
### 网卡配置文件：
### /etc/sysconfig/network-scripts/ifcfg-ens33
### DNS配置文件:
### /etc/resolv.conf
### 网络主机绑定IP地址：
### /etc/hosts
### 计算机名：
### /etc/hostname
### 查看内存使用情况 free -m  
### 查看cpu使用情况 top   
### 查看磁盘以及分区情况  df -h   
### 查看端口使用情况     lsof -i:端口号     netstat -apn|grep 端口号 ps -au|grep 端口号

### chkconfig --list 列出所有的服务
### chkconfig  服务名称	[on/off]开机启动/开机不启动
### service 服务名成 [start/stop/restart]

### centos7服务管理
systemctl //列出正在运行的服务
systemctl list-unit-files //所有已经安装的服务
systemd-cgls   以树形列出正在运行的进程，它可以递归显示控制组内容
systemctl start postfix.service  启动一个服务
systemctl stop postfix.service	停止
systemctl restart postfix.service	重启
systemctl status postfix.service	查看服务的状态
systemctl enable postfix.service	设置开机启动
systemctl disable postfix.service	设置开机不启动
systemctl is-enabled postfix.service	查看是否开机启动
### which 命令	查看指定命令对应程序所在的位置，类似于查看快捷方式对应的实际文件
### yum list installed  //查看所有已经安装的程序  
### yum  localinstall docker-engine-selinux-1.12.6-1.el7.centos.noarch.rpm  --nogpgcheck  //本地安装
### yum grouplist //列出所有的软件组 比如 genome desktop
### yum group install 某个组的名称 //安装某个组的程序，一系列程序
### yum --downloadonly --downloaddir ./download  //把指定的软件下载到本机目录，不进行安装 试用单个程序或者组
### 局域网共享yum install samba --downloadonly --downloaddir ./download
yum install java-1.8.0-openjdk-devel.x86_64 --downloadonly --downloaddir /root/jdk1.8
yum install java-11-openjdk-devel.x86_64 --downloadonly --downloaddir /root/jdk11
java-11-openjdk-devel.x86_64
### 映射网络驱动器 mount -t cifs -o username="administrator",password="xxx" //192.168.56.101/Downloads /LFIS_Release

###挂在windows的共享 使用smbfs文件系统 mount -t smbfs -o username=xxx,password=xxx,-l //192.168.56.1/Downloads /mnt/hostDownloads
###挂在windows的共享 使用cifs文件系统 mount -t cifs -o username="xxx",password="xxx" //192.168.56.1/Downloads /mnt/Downloads/ 
###安装文件系统 install cifs-utils

重启系统的时候自动mount, 将下面命令行添加到/etc/fstab里。
//192.168.56.1/Downloads /mnt/downloads/ cifs defaults,username=Deroom,password=BT151 0 2 

### yum list available
### uname -r
### pwd 当前所在的位置 printf   working directory
### /bin 命令对应的执行程序
### /dev    设备被抽象成文件
### /etc    系统的配置文件，第三方程序的配置信息
### /home   所有用户的目录
### /lib    动态类库，
### /media  自动挂载
### /mnt    手动挂在
### /root   管理员的一个目录
### /usr    当前用户的软件安装目录
### ls -a列出所有  -l详细信息 -R递归子目录
### 创建目录 mkdir aa/bb/cc -p表示多级目录
### 创建文件    touch
### 删除目录 rmdir 只能删除空目录
### 删除目录或者文件    rm -r表示递归
### 复制命令 cp 源文件/目录 目标文件/目录    -r递归操作（目录情况下）
### 查看文件内容    cat more    less    head    tail
### 修改文件名/文件夹名  mv 源路径 目标路径

### 创建软链接（快捷方式，指向一个硬链接）    ln -s 源文件/源目录    目标路径
### 创建硬链接（并不占用磁盘空间，链接到磁盘地址） ln 源文件 目标路径
### 硬链接相关，查看Inode   stat 文件
### 文件或目录属性  wc 查看文本文件的行数，单词书，字节数   od 查看二进制文件
### du -h   查看目录占用

### 文件的查找和检索，按文件属性    find 路径 -name（按名称查） 文件名称  (通配符*一个或多个字符，?一个字符) -size(按大小查找) + 10k(大于10k) - 10M（小于10M）  如果按大小范围就用两个size -type(安装文件类型查)
### 文件的查找和检索，按照文件内容  grep -r递归 "查找内容" "查找路径"
###文件详细信息解释  d(文件类型)rwx(所有者权限)r-x(所在组权限)r-x(其他用户权限)  7（硬链接数量）  root(所属用户) root(所在的组) 4096（大小）   
### drwxr-xr-x. 7 root   root      4096 Jul  3  2018 dotnet
### 文件类型    普通文件-   目录d   链接符号l   块设备b 字符设备c   socket文件s 管道p

### 修改文件权限 chmod  777 文件或者目录
### 修改所在的组    chown 文件所有者【：文件所属组】 文件或者目录

### 压缩和解压缩    简易版 gzip *.txt ,gunzip a.txt.gz .gz格式的压缩包;  bzip2 .bz2格式压缩包
### 压缩和解压缩    高阶版  tar zcvf xxx.tar.gz *.txt  jcvf xxx.tar.bz2 *.txt  参数解释：c压缩 x解压缩 v显示提示信息 f指定压缩文件的名字 z使用gz的方式压缩 j使用bzip2方式压缩 -C压缩到指定目录，解压缩到指定目录 默认到当前目录

## 进程管理
### ps a当前操作系统的所有用户 u显示用户自己的信息 x没有终端的应用程序 ps aux | grep bash 利用管道检索指定的进程
### who查看当前用户 tty1-tty6文字终端 tty7带桌面的终端 ctrl+alt+f1-f7进行切换 pts/0设备终端
### kill结束进程 -信号 -pid -l列出所有信号
### env 环境变量 env | grep PATH
### top 任务管理器  

## 网络管理
### ifconfig    ping  地址  -c次数      nslookup www.baidu.com

## 用户管理
### 创建用户 adduser 用户名     --这个本质是个脚本，后续会提示设置密码等步骤，非常方便
### 创建用户    useradd  很多参数
### 删除用户    deluser 用户名
### 删除用户    userdel -r 用户名   --这个会把用户对于的桌面文件夹删掉
### 添加用户组  groupadd 组名
### 修改密码    passwd  用户名
### 直接切换用户 su 用户名    
### 查看所有的用户  利用配置文件查看    /etc/passwd
### ssh 用户名@ip 基于服务器openssh-server
### logout  登出
### scp(super copy  基于ssh)
## 挂载 mount  挂载U盘，挂载FTP

## 帮助文档 man man
##  输出指定的字符  echo $PATH      $后面跟一个变量
## 开关机   halt 立刻关机  poweroff 立刻关机  shutdown -h now 立刻关机(root用户使用) shutdown -r now 立刻重启(root用户使用)

##  静态库制作
### 命名规则：lib+库名字+.a ,例如：libazeroth.a
### 制作步骤：1、生成对应的.o文件，gcc ./src/*.c -c -I ./include 2、把.o文件打包，ar rcs 静态库名字(libazeroth.a) 所有的.o
### 发布和使用静态库：.a文件和头文件， gcc main.c ./lib/libazeroth.a -I ./include (等价写法：gcc main.c -I ./include -L lib -l azeroth)
### 查看.a文件的内容：nm ./lib/libazeroth.a             nm ./a.out
### 静态库优点，发布应用程序时，不需要提供.a文件（.a里面的.o内容已经被编译到应用程序中），加载库的速度快
### 静态库缺点，库被打包到应用程序中，导致应用程序体积较大，如果库升级，应用程序也需要升级

##  动态库制作
### 命名规则：lib+库名字+.so,例如：libazeroth.so
### 制作步骤：1、生成.o文件（与位置无关的.o文件，程序运行时加载，所以所有的地址都是相对地址，加载位置后的偏移量），gcc ./src/*.c -c -fPIC -I ./include      与静态库相比，增加参数 -fPIC 或者-fpic,  2、打包生成.so文件，gcc *.o -shared -I ./include -o libazeroth.so       这个和静态库完全不一样
### 发布和使用动态库：.so文件和头文件， gcc main.c -I ./include ./lib/libazeroth.so -o app（等价写法gcc main.c -I ./include -L lib -l azeroth -o myapp  这样会发生动态库链接失败）,动态库链接失败解决办法，1、把库放到/lib里面，2、export LD_LIBRARY_PATH=./lib,这是把当前的lib目录添加到环境变量，这个修改是临时的，基于当前会话，3、修改当前bash的配置文件（这个不推荐）  4、修改动态链接器的配置文件，vi /etc/ld.so.conf 增加一行/root/itcast/t4/lib 这个动态库的路径，更新配置ldconfig -v
### 动态连接器 ldd 应用程序    查看应用程序依赖的所有动态库，动态链接器会自动加载依赖的动态库，按照环境变量里的路径找，
### 优缺点，类似，应用程序需要包含so.文件

### 环境变量 临时修改某个键的值 export LD_LIBRARY_PATH=./lib
###	export PATH="/tmp:$PATH"  //这个的意思是，重新给环境变量的PATH赋值，$PATH表示原来的PATH值，PATH值用:分割
###	永久修改环境变量的值 第一步，修改文件/etc/profile，在末尾加上行 PATH="/tmp:$PATH"，第二步，source /etc/profile 

### gdb调试

### Makefile或者makefile ,makefile的规则，三要素：目标，依赖，命令
### 说明，例子里的所有文件在同一个目录
目标:依赖
    命令
原始版本
app:main.c add.c divid.c
	gcc main.c add.c divid.c -o app
改进版，拆分为子目标
app:main.o add.o divid.o
	gcc main.o add.o divid.o -o app

main.o:main.c
	gcc -c main.c

add.o:add.c
	gcc -c add.c

divid.o:divid.c
	gcc -c divid.c

makefile中的自动变量（只能在命令中使用）
$<	规则中的第一个依赖
$@	规则中的目标
$^	规则中的所有依赖
% 模式匹配

改进版（自定义变量，不需要类型，使用变量：$(变量名称)，使用模式匹配和自动变量）
obj=main.o add.o divid.o
target=app
$(target):$(obj)
	gcc $^ -o $@

%.o:%.c
	gcc -c $< -o $@

makefile中的默认的系统变量，都是大写
CC	值为cc
CPPFLAGS	预编译的时候需要的选项 例如 -I
CFLAGS	编译时使用的参数 例如 -Wall -g -c
LDFLAGS	链接库时使用的选项	-L -l

makefile中的函数
获取指定目录下的.c文件	src=$(wildcard ./*.c)
匹配替换	obj=$(patsubst ./%.c,./%.o,$(src))  这样获取.c对于的.o

改进版 使用函数
src=$(wildcard ./*.c)
obj=$(patsubst ./%.c,./%.o,$(src))
target=app
$(target):$(obj)
	gcc $^ -o $@

%.o:%.c
	gcc -c $< -o $@

clean:
	rm $(obj) $(target) -f

hello:
	echo $(src),$(obj),$(LDFLAGS),$(CC)

make默认生成第一个目标，make clean 这是指定make的目标


# gtk开发
## yum install libgnomeui-devel

# hadoop安装，vbox新建三天机器后，centos7会自动配置网卡，比centos6方便，centos6需要更新网卡配置文件里的mac地址才能访问网络
# 准备三台机器，hadoopNameNode,hadoopDataNode1,hadoopNameNode2
# 修改机器的名称 /etc/hostname
# 修改机器的hosts文件 /etc/hosts
#	安装jdk1.8，yum localinstall以后不会设置环境变量，因为已经配置的PATH，所有可以直接运行java和javac，但是仍然需要设置相关环境变量，其他程序依赖这个这几个环境变量的值
#JDK全局环境变量配置
export JAVA_HOME=/usr/java/jdk1.8.0_231-amd64
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
# 把hadoop安装程序上传到hadoopNameNode ，目录/export/server/,直接解压到当前目录，tar zxvf hadoop-2.7.4.tar.gz
# 配置文件修改
#/export/server/hadoop2.7.4/etc/hadoop//hadoop-env.sh，找到 export JAVA_HOME={JAVA_HOME}，修改为export JAVA_HOME=/usr/java/jdk1.8.0_231-amd64
#/export/server/hadoop2.7.4/etc/hadoop/core-site.xml，
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://hadoopNameNode:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/home/hadoop2.7.4_data</value>
	</property>
</configuration>

#hdfs-site.xml，
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
	<property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>hadoopDataNode1:50090</value>
	</property>
</configuration>
# mv mapred-site.xml.template mapred-site.xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>

#yarn-site.xml
<configuration>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>hadoopNameNode</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
</configuration>
#slaves
hadoopNameNode
hadoopDataNode1
hadoopDataNode2

#把hadoop的程序路径配置到环境变量
export HADOOP_HOME=/export/server/hadoop2.7.4
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

#虚拟机内存不要低于2048，512和1024MB的情况下，hadoop可以正常启动起来，但是执行demo程序，resourcemanager就出错终止运行了

# 下发配置好的文件到其他机器 scp -r /export/server/hadoop2.7.4 root@hadoopDataNode1:/export/server/hadoop2.7.4/
scp -r /export/server/hadoop2.7.4 root@hadoopDataNode2:/export/server/hadoop2.7.4/
# 下发环境变量配置文件
scp -r /etc/profile root@hadoopDataNode2:/etc/
scp -r /etc/profile root@hadoopDataNode1:/etc/

# 刷新环境变量 source /etc/profile

#配置文件说明
***-default.xml 这里面配置了hadoop默认的配置选项
如果用户没有修改，那么这里面的选项将会生效

***-site.xml 这里面配置了用户需要自定义的配置选项

site中配置的值优先级大于default中的配置项的值

#启动hadoop集群，需要启动HDFS集群，YARN集群
首次启动HDFS集群，需要对其进行格式化（初始化），格式化只能进行一次，和yarn没有关系,集群启动成功以后，不要再进行格式化
在namenode所在的机器上进行hdfs格式化
格式化命令：hdfs namenode -format，执行后需要等待

#脚本启动hdfs集群，也可以单点一个个启动，
#前提条件，namenode到其他机器免密登陆，配置文件slaves配置好了
脚本：/export/server/hadoop2.7.4/sbin/start-dfs.sh，也有配套的停止脚本
#脚本启动yarn集群
脚本：/export/server/hadoop2.7.4/sbin/start-yarn.sh

#查看状态
jps

#单节点逐个启动

#集群启动成功后，提供web查看
http://192.168.56.61:50070
http://192.168.56.61:8088

#执行一个mapreduce
hadoop jar  hadoop-mapreduce-examples-2.7.4.jar pi 20 50



#windows servercore
	映射网络上共享目录到指定盘符：net use Z: \\192.168.56.1\Downloads BT151 /user:Deroom
	删除指定的映射	net use Z: /del 
	打开 WoW64：Start /w ocsetup ServerCore-WOW64
  打开 .NET 2.0 层：Start /w ocsetup NetFx2-ServerCore
  打开 WoW64 的 .NET 2.0 层: Start /w ocsetup NetFx2-ServerCore-WOW64
	安装servercore版net40
	安装powershell DISM /Online /Enable-Feature /FeatureName:MicrosoftWindowsPowerShell 
	
	升级到ps3.0 依赖net40 ,ps4.0依赖net45
	禁用虚拟内存
	HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\DisablePagingExecutive
	设置为1
	删除或者设置虚拟内存文件
	先wmic进入交互模式（必须），然后
	computersystem where name="WIN-TAJ0QBHFCHC" set AutomaticManagedPagefile=False
	重启后
	PageFileSet where "name='C:\\pagefile.sys'" delete
	禁用休眠
	powercfg -h off 重启后生效，会自动删除休眠文件
	关闭防火墙
	netsh firewall set opmode disable
netsh advfirewall firewall add rule name=sshd dir=in action=allow protocol=TCP localport=22
netsh advfirewall firewall set opmode disable

	计算机管理-远程
	管理机配置凭据，windows和普通凭据，具体用哪个地方的待研究，
	计算机管理连接192.168.56.101，就可以进行管理

	配置sqlserverexpress，监听tcp1433
	

	登陆：BT@151
	数据库：hw@911226

	安装IIS
	dism /online /enable-feature /featurename:IIS-WebServerRole
    dism /online /enable-feature /featurename:IIS-ISAPIFilter
    dism /online /enable-feature /featurename:IIS-ISAPIExtensions
    dism /online /enable-feature /featurename:IIS-NetFxExtensibility
		安装IIS-ASPNET
    dism /online /enable-feature /featurename:IIS-ASPNET
		
		通用 IIS 命令行管理工具。
		appcmd管理iis,AppCmd.exe is located in the %systemroot%\system32\inetsrv\ directory
		总的命令格式：APPCMD (命令) (对象类型) <标识符> </参数1:值1 ...>
		查看所有的参数：appcmd list sites /text:*
		appcmd list apppool /text:*
		修改site的应用程序池
		appcmd set site "Default web site" -[path='/'].applicationPool:"ASP.NET v4.0"
		查看某个对象的操作参数
		appcmd set site "Default web site" /?

		ie打不开，修改注册表权限
		HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\Main右键点击Main，选择权限，启用继承

		111






	











