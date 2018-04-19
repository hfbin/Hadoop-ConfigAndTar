# 从零开始配置hadoop集群环境
<h5>一张图了解hadoop集群环境架构</h5>

![这里写图片描述](http://img.blog.csdn.net/20171223134751933?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
（其它废话我就不多说了，直接上）

这里在VMware装liunx（rhel-server-6.4，一定要注意是liunx版本是6的）虚拟机我就不多说了，需要的留言，有必的话留要的话我会另写一篇文章在VMware装liunx虚拟机。
 
#### **1.配置网络和计算机名字**
##### **1) 关闭防火墙和selinux**
	shell>iptables -F
	shell>service iptables save
	shell>setenforce 0
	shell>vim /etc/selinux/config 修改SELINUX=disabled
##### **2) 配置计算机网络**
	shell>vim /etc/sysconfig/network-scripts/ifcfg-eth0
内容如下：加//的表示要修改的地方 没有的自己加进去
>DEVICE=eth0
HWADDR=00:0C:29:57:A1:42
TYPE=Ethernet
ONBOOT=yes //
NM_CONTROLLED=no//关闭，yes表示修改后不必重启网卡立刻生效。会带来隐患
BOOTPROTO=static	//静态分配ip，不要用DHCP分配
IPADDR=172.16.100.101 //
GATEWAY=172.16.100.2 //

重启网卡

	shell>service network restart 
测试：查看ip

	shell>ifconfig 查看ip地址
如图：
![这里写图片描述](http://img.blog.csdn.net/20171223110550365?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
此时你在本地ping虚拟机时候还是ping不通的，看下一步。
##### **3) 配置vmware的网卡**
![这里写图片描述](http://img.blog.csdn.net/20171223110633295?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20171223110646735?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里网关ip必须与/etc/sysconfig/network-scripts/ifcfg-eth0中的GATEWAY的ip一致。

![这里写图片描述](http://img.blog.csdn.net/20171223110655801?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果本虚拟机是克隆的，查看一下配置的mac地址和虚拟机分配的mac地址是否一致

![这里写图片描述](http://img.blog.csdn.net/20171223110705439?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里如果还是ping不同那就要到本地网络里去重启一下VMnet8网络。

NAT方式如果仍然上不去网另留言。

##### **4) 配置计算机名字**
临时修改
  
	shell>hostname hfbin1  
永久修改（需要重启,如果执行了临时修改，不需要重启）

	shell>vim /etc/sysconfig/network 


#### **2 安装JDK和Hadoop**
JDK和Hadoop包百度云链接：https://pan.baidu.com/s/1pLEAAa3 密码：gymn
##### **2.1	安装jdk1.7.0.71**
**卸载旧的jdk**

    shell>rpm -qa | grep jdk
执行命令显示内容如下：

java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.i686

java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.i686

卸载jdk:

    shell>rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.50.1.11.5.el6_3.i686
    shell>rpm -e --nodeps java-1.7.0-openjdk-1.7.0.9-2.3.4.1.el6_3.i686

如图：
 ![这里写图片描述](http://img.blog.csdn.net/20171223110954031?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
**2、	安装新的jdk，要求1.7以上版本(从现在开始我使用xshell工具链接虚拟机不用进入虚拟机里面操作了)。**

这里要装VMware tool 工具才能将文件往虚拟机拖，不过上面你要是网络配通了，可以使用winscp/xftp工具将文件上传到虚拟机上，在这我就不说明了，这里我将文件上传到了/opt目录中。

进入opt目录

    shell>cd /opt

执行安装：

    shell>rpm –ivh jdk-7u67-linux_64.rpm
   ![这里写图片描述](http://img.blog.csdn.net/20171223111035444?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
**3、	配置jdk环境变量**
修改/etc/profile
shell> vim  /etc/profile
在末尾加入：

    export JAVA_HOME=/usr/java/jdk1.7.0_67
    export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
    export PATH=.:$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
 ![这里写图片描述](http://img.blog.csdn.net/20171223111118661?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
进入终端，输入 source /etc/profile 使刚刚修改的环境变量生效。

    shell>source /etc/profile

测试是否成功：

    shell>java –version //1.7.0.67
    shell>javac –version //1.70.67
 ![这里写图片描述](http://img.blog.csdn.net/20171223111129827?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
##### **2.2	创建用户hduser和组hadoop**

    shell>groupadd hadoop
    shell>useradd –g hadoop hduser 
![这里写图片描述](http://img.blog.csdn.net/20171223111139851?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    shell>passwd hduser
 ![这里写图片描述](http://img.blog.csdn.net/20171223111152375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
修改vim /etc/sudoers,为hduser增加sudo权限

    shell>sudo vim /etc/sudoers
    
输入密码，使用root权限才可以更改此文件。

大概在99行，添加内容：

    hduser  ALL=(ALL)  ALL  (大概99行)
 ![这里写图片描述](http://img.blog.csdn.net/20171223111205468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

重启 

    shell>reboot

重启，切换到hduser登陆


##### **2.3	安装hadoop2.6.0**

以后的身份用hduser，如果涉及到权限不够，用sudo

先通过vmtools或winscp/xftp把hadoop-2.6.0.tar.gz传到linux的/home/hduser中

解压:

shell>tar xvzf hadoop-2.6.0.tar.gz 

改名:

shell>mv hadoop-2.6.0   hadoop-2.6

##### **2.4	配置hadoop的环境变量**
shell>vim /etc/profile

在末尾加入下面环境：

export HADOOP_HOME=/home/hduser/hadoop-2.6
export PATH=.:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:\$PATH

进入终端，输入 shell>source /etc/profile 使刚刚修改的环境变量生效

测试hadoop是否安装成功

shell>hadoop version

如图：

![这里写图片描述](http://img.blog.csdn.net/20171223113600820?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

看到图片信息即可安装成功。



#### **3	安装ssh**
1）	生成公钥和私钥

       shell>ssh-keygen  -t rsa 		//在 /home/hduser/.ssh/目录下有两把钥匙，其中后缀.pub是公

2）	在本机进行免密码登陆

    shell>ssh-copy-id localhost			//复制私钥到其它计算机，当前是本机 
                                             //一会克隆完其它两台机器，还要再用此命令。

测试，输入：

	shell>ssh localhost

如果不需要密码则正确


#### **4	配置完全分布式**
##### **4.1	配置host映射文件**
修改shell>sudo vim /etc/hosts文件，做ip和主机名映射

    172.16.100.101 hfbin1
    172.16.100.102 hfbin2
    172.16.100.103 hfbin3

修改本机的主机名为hfbin1

    shell>hostname hfbin1 			//临时修改，立刻起作用
    shell>vim /etc/sysconfig/network	//永久修改，需要重启

##### **4.2	克隆两台虚拟机,修改主机名和ip(注意mac地址)**
**1） 克隆两台虚拟机，分别是hfbin2和hfbin3**
 ![这里写图片描述](http://img.blog.csdn.net/20171223130621247?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
然后一直点下一步，下一步，然后选择“完整克隆”，之后选择存放目录，再点完成。
  ![这里写图片描述](http://img.blog.csdn.net/20171223130659445?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
**2)修改主机(ip和mac地址)**
  
主机：

    shell>sudo vim /etc/sysconfig/network ，之后重启（不想重启需要再执行命令hostname）
	hostname=hfbin2（hfbin3）
	同时删除；NETWORKING=yes
	
ip:  

    shell>sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0
	IPADDR=172.16.100.102(172.16.100.103)
    HWADDR=？ 见下图

mac: 还是在上面的文件中修改mac地址，要和vmware分配的一样。
 ![这里写图片描述](http://img.blog.csdn.net/20171223130723053?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
克隆后的虚拟机，如果ip没有发现，先重启，还不行，执行以下步骤：

>1、	删除文件 shell>rm -rf /etc/udev/rules.d/70-persistent-net.rules

>2、	修改hwaddr为实际的mac地址。shell>vim /etc/sysconfig/network-scripts/ifcfg-eth0

>3、	 shell>reboot重启

##### **4.3	设置ssh免登陆到其它主机**

把hfbin1的公钥复制到hfbin2和 hfbin3中，这样hfbin1可以免登录到其它两台主机上。

    shell>ssh-copy-id	hfbin2
    shell>ssh-copy-id	hfbin3

测试，在hfbin1输入：

	shell>ssh hfbin2
	shell>ssh hfbin3

如果不需要密码则正确（注意进入hfbin2或hfbin3想退出可以用shell>exit ,即可）

##### **4.4	修改hadoop的七个配置文件**

这里只对hfbin1进行更改，hfbin2和hfbin3等一下直接copy进去即可。

以下7个文件在hadoop的安装目录的etc下，也就是~/hadoop-2.6/etc/hadoop下

1)Hadoop的环境配置，我们主要配置一下JDK路径。

    shell>vim hadoop-env.sh
修改下面内容

    export JAVA_HOME=/usr/java/jdk1.7.0_67
 ![这里写图片描述](http://img.blog.csdn.net/20171223130801918?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
2)Yarn框架环境配置，也要指定JDK路径。
    
    shell>vim yarn-env.sh
修改下面内容

    export JAVA_HOME=/usr/java/jdk1.7.0_67
 ![这里写图片描述](http://img.blog.csdn.net/20171223130816324?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
3)	增加slave节点，也就是DateNode节点。

    shell>vim slaves
删除所有，加入下面内容

    hfbin2
    hfbin3	
 
4)Hdoop的全局配置。例如：客户端连接HDFS时，路径前缀和工作端口。

    shell>vim core-site.xml
在configuration标签里加入下面内容：
```xml
<configuration>
	  <property>
		<name>fs.defaultFS</name>
		<value>hdfs://hfbin1:9000</value>
	  </property>
	  <property>
		<name>hadoop.tmp.dir</name>
		<value>file:/home/hduser/hadoop-2.6/tmp</value>
	  </property>
</configuration>
```
 
5)HDFS的配置

    shell>vim hdfs-site.xml
在configuration标签里加入下面内容：
```xml
<configuration>
	<property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>hfbin1:50090</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:/home/hduser/hadoop-2.6/dfs/name</value>
	</property>
	<property>
		<name>dfs.datanode.data.dir</name>
		<value> file:/home/hduser/hadoop-2.6/dfs/data</value>
	</property>
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
	<property>
		<name>dfs.webhdfs.enabled</name>
		<value>true</value>
	</property>
</configuration>
```
 
6)Mapreduce的配置

    shell>vim mapred-site.xml
在configuration标签里加入下面内容：
```xml
<configuration>
	  <property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	  </property>
	  <property>
		<name>mapreduce.jobhistory.address</name>
		<value>hfbin1:10020</value>
	  </property>
	  <property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value> hfbin1:19888</value>
	  </property>
</configuration>
```
 
7)Yarn框架的配置Hadoop集群测试

    shell>vim yarn-site.xml
在configuration标签里加入下面内容：
```xml
<configuration>

<!-- Site specific YARN configuration properties -->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
		<value>org.apache.hadoop.mapred.ShuffleHandler</value>
	</property>
	<property>
		<name>yarn.resourcemanager.address</name>
		<value>hfbin1:8032</value>
	</property>
	<property>
		<name>yarn.resourcemanager.scheduler.address</name>
		<value>hfbin1:8030</value>
	</property>
	<property>
		<name>yarn.resourcemanager.resource-tracker.address</name>
		<value>hfbin1:8035</value>
	</property>
	<property>
		<name>yarn.resourcemanager.admin.address</name>
		<value>hfbin1:8033</value>
	</property>
	<property>
		<name>yarn.resourcemanager.webapp.address</name>
		<value>hfbin1:8088</value>
	</property>
</configuration>
```
复制7个文件到hfbin2和hfbin3

    shell>scp -r /home/hduser/hadoop-2.6/etc/hadoop/ hduser@hfbin2:/home/hduser/hadoop-2.6/etc/
    shell>scp -r /home/hduser/hadoop-2.6/etc/hadoop/ hduser@hfbin3:/home/hduser/hadoop-2.6/etc/

##### **4.5	验证hadoop配置是否正确**
**1、	格式化master(namenode)的文件系统**

    shell> hdfs namenode –format
    
注：如果已经格式化后，重新格式化，需要删除172.16.100.101、172.16.100.102、172.16.100.103节点的此目录下的文件：/home/hduser/hadoop-2.6/dfs

**2、	启动dfs**

    shell>start-dfs.sh
![这里写图片描述](http://img.blog.csdn.net/20171223131225949?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果出现WARN util.NativeCodeLoader: Unable：： to load native-hadoop library….

解决：在shell>vim /home/hduser/hadoop-2.6/etc/hadoop/log4j.properties文件中末尾添加

    log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR

3、	输入 jps查看进程

    shell>jps
![这里写图片描述](http://img.blog.csdn.net/20171223131322168?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
4、	启动yarn

    shell>start-yarn.sh
![这里写图片描述](http://img.blog.csdn.net/20171223131357970?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
5、	查看集群状态

     shell>hdfs dfsadmin -report
 ![这里写图片描述](http://img.blog.csdn.net/20171223131535051?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
6、	在浏览中查看dfs的运行状态

    http://172.16.100.101:50070
![这里写图片描述](http://img.blog.csdn.net/20171223131742714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
7、	停止hadoop

    shell>stop-all.sh
    
    
##### **4.6	运行例子wordcount**
该例子为hadoop自带例子，统计出所有文件的单词的词频。

如图：
![这里写图片描述](http://img.blog.csdn.net/20171223134322221?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
1)	创建文件file1和file2(放到目录/home/hduser/file)
    
    shell>mkdir file (在/home/hduser/下输入命令)
    shell>vim file1
    加入下面内容：
    Hello World hi HADOOP
    shell>vim file2
    加入下面内容：
    Hello Hadoop hi CHINA
    
2)	启动hdfs，并且创建hdfs目录 /input3

    shell>start-dfs.sh
    shell>start-yarn.sh
    shell>hadoop  fs  -mkdir  /input3
3)	把file1和file2放到hdfs的目录/input2/中。

    shell> hadoop fs  -put file* /input3
4)	查看hdfs中是否有file和file2

    shell>hadoop fs  -ls /input3
![这里写图片描述](http://img.blog.csdn.net/20171223133710586?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
5)	运行命令
    
    shell>hadoop jar /home/hduser/hadoop-2.6/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar wordcount  /input3/ /output3/wordcount1
6)	查看输出文件

    shell>hadoop fs -ls /output3/wordcount1
![这里写图片描述](http://img.blog.csdn.net/20171223133951715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
   
    shell>hadoop fs -cat /output3/wordcount1/part-r-00000

![这里写图片描述](http://img.blog.csdn.net/20171223134008623?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzM1MjQxNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



后续我会写一篇关于**hdfs命令**的使用方法





<h3>目前到这hdoop集群搭建就完毕了</h3>有什么问题在下面留言即可
