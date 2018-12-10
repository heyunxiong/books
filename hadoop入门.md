尚硅谷大数据技术之Hadoop（入门）

(作者：大海哥)

 

官网：[www.atguigu.com](http://www.atguigu.com)

版本：V1.2

# 一 从Hadoop框架讨论大数据生态

## 1.1 Hadoop是什么

1）Hadoop是一个由Apache基金会所开发的分布式系统基础架构

2）主要解决，海量数据的存储和海量数据的分析计算问题。

3）广义上来说，HADOOP通常是指一个更广泛的概念——HADOOP生态圈

![img](D:\software\Typora\imgs\clip_image002-1544403002406.jpg)

## 1.2 Hadoop发展历史

1）Lucene--Doug Cutting开创的开源软件，用java书写代码，实现与Google类似的全文搜索功能，它提供了全文检索引擎的架构，包括完整的查询引擎和索引引擎 

2）2001年年底成为apache基金会的一个子项目

3）对于大数量的场景，Lucene面对与Google同样的困难

4）学习和模仿Google解决这些问题的办法 ：微型版Nutch

5）可以说Google是hadoop的思想之源(Google在大数据方面的三篇论文)

​       GFS --->HDFS

​       Map-Reduce --->MR

​       BigTable --->Hbase

6）2003-2004年，Google公开了部分GFS和Mapreduce思想的细节，以此为基础Doug Cutting等人用了2年业余时间实现了DFS和Mapreduce机制，使Nutch性能飙升 

7）2005 年Hadoop 作为 Lucene的子项目 Nutch的一部分正式引入Apache基金会。2006 年 3 月份，Map-Reduce和Nutch Distributed File System (NDFS) 分别被纳入称为 Hadoop 的项目中 

8）名字来源于Doug Cutting儿子的玩具大象![img](D:\software\Typora\imgs\clip_image004.jpg)

9）Hadoop就此诞生并迅速发展，标志这云计算时代来临

## 1.3 Hadoop三大发行版本

Hadoop 三大发行版本: Apache、Cloudera、Hortonworks

 Apache版本最原始（最基础）的版本，对于入门学习最好。

Cloudera在大型互联网企业中用的较多。

Hortonworks文档较好。

1）Cloudera Hadoop 

​        （1）2008年成立的Cloudera是最早将Hadoop商用的公司，为合作伙伴提供Hadoop的商用解决方案，主要是包括支持、咨询服务、培训。

​        （2）2009年Hadoop的创始人Doug Cutting也加盟Cloudera公司。Cloudera产品主要为CDH，Cloudera Manager，Cloudera Support

​        （3）CDH是Cloudera的Hadoop发行版，完全开源，比Apache Hadoop在兼容性，安全性，稳定性上有所增强。

​        （4）Cloudera Manager是集群的软件分发及管理监控平台，可以在几个小时内部署好一个Hadoop集群，并对集群的节点及服务进行实时监控。Cloudera Support即是对Hadoop的技术支持。

​        （5）Cloudera的标价为每年每个节点4000美元。Cloudera开发并贡献了可实时处理大数据的Impala项目。

2）Hortonworks Hadoop

​        （1）2011年成立的Hortonworks是雅虎与硅谷风投公司Benchmark Capital合资组建。

​        （2）公司成立之初就吸纳了大约25名至30名专门研究Hadoop的雅虎工程师，上述工程师均在2005年开始协助雅虎开发Hadoop，贡献了Hadoop80%的代码。

​        （3）雅虎工程副总裁、雅虎Hadoop开发团队负责人Eric Baldeschwieler出任Hortonworks的首席执行官。

​        （4）Hortonworks的主打产品是Hortonworks Data Platform（HDP），也同样是100%开源的产品，HDP除常见的项目外还包括了Ambari，一款开源的安装和管理系统。

​        （5）HCatalog，一个元数据管理系统，HCatalog现已集成到Facebook开源的Hive中。Hortonworks的Stinger开创性的极大的优化了Hive项目。Hortonworks为入门提供了一个非常好的，易于使用的沙盒。

​        （6）Hortonworks开发了很多增强特性并提交至核心主干，这使得Apache Hadoop能够在包括Window Server和Windows Azure在内的microsoft Windows平台上本地运行。定价以集群为基础，每10个节点每年为12500美元。

## 1.4 Hadoop的优势

1）高可靠性：因为Hadoop假设计算元素和存储会出现故障，因为它维护多个工作数据副本，在出现故障时可以对失败的节点重新分布处理。

2）高扩展性：在集群间分配任务数据，可方便的扩展数以千计的节点。

3）  高效性：在MapReduce的思想下，Hadoop是并行工作的，以加快任务处理速度。

4）高容错性：自动保存多份副本数据，并且能够自动将失败的任务重新分配。

## 1.5 Hadoop组成

1）Hadoop HDFS：一个高可靠、高吞吐量的分布式文件系统。

2）Hadoop MapReduce：一个分布式的离线并行计算框架。

3）Hadoop YARN：作业调度与集群资源管理的框架。

4）Hadoop Common：支持其他模块的工具模块。![img](D:\software\Typora\imgs\clip_image006-1544403002407.png)

### 1.5.1 HDFS架构概述

1）NameNode（nn）：存储文件的元数据，如文件名，文件目录结构，文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和块所在的DataNode等。

![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg) ![img](D:\software\Typora\imgs\clip_image010-1544403002408.jpg)

2）DataNode(dn)：在本地文件系统存储文件块数据，以及块数据的校验和。

![img](D:\software\Typora\imgs\clip_image012.jpg) ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image014.jpg)

3）Secondary NameNode(2nn)：用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

### 1.5.2 YARN架构概述

1）ResourceManager(rm)：处理客户端请求、启动/监控ApplicationMaster、监控NodeManager、资源分配与调度；

2）NodeManager(nm)：单个节点上的资源管理、处理来自ResourceManager的命令、处理来自ApplicationMaster的命令；

3）ApplicationMaster：数据切分、为应用程序申请资源，并分配给内部任务、任务监控与容错。

4）Container：对任务运行环境的抽象，封装了CPU、内存等多维资源以及环境变量、启动命令等任务运行相关的信息。

### 1.5.3 MapReduce架构概述

MapReduce将计算过程分为两个阶段：Map和Reduce

1）Map阶段并行处理输入数据

2）Reduce阶段对Map结果进行汇总

## 1.6 大数据技术生态体系

![img](D:\software\Typora\imgs\clip_image016-1544403002408.png)

图中涉及的技术名词解释如下：

1）Sqoop：sqoop是一款开源的工具，主要用于在Hadoop(Hive)与传统的数据库(mysql)间进行数据的传递，可以将一个关系型数据库（例如 ： MySQL ,Oracle 等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

2）Flume：Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。

3）Kafka：Kafka是一种高吞吐量的分布式发布订阅消息系统，有如下特性：

（1）通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时间的稳定性能。

（2）高吞吐量：即使是非常普通的硬件Kafka也可以支持每秒数百万的消息

（3）支持通过Kafka服务器和消费机集群来分区消息。

（4）支持Hadoop并行数据加载。

4）Storm：Storm为分布式实时计算提供了一组通用原语，可被用于“流处理”之中，实时处理消息并更新数据库。这是管理队列及工作者集群的另一种方式。 Storm也可被用于“连续计算”（continuous computation），对数据流做连续查询，在计算时就将结果以流的形式输出给用户。

5）Spark：Spark是当前最流行的开源大数据内存计算框架。可以基于Hadoop上存储的大数据进行计算。

6）Oozie：Oozie是一个管理Hdoop作业（job）的工作流程调度管理系统。Oozie协调作业就是通过时间（频率）和有效数据触发当前的Oozie工作流程。

7）Hbase：HBase是一个分布式的、面向列的开源数据库。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。

8）Hive：hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。

10）R语言：R是用于统计分析、绘图的语言和操作环境。R是属于GNU系统的一个自由、免费、源代码开放的软件，它是一个用于统计计算和统计制图的优秀工具。

11）Mahout:

Apache Mahout是个可扩展的机器学习和数据挖掘库，当前Mahout支持主要的4个用例：

推荐挖掘：搜集用户动作并以此给用户推荐可能喜欢的事物。

聚集：收集文件并进行相关文件分组。

分类：从现有的分类文档中学习，寻找文档中的相似特征，并为无标签的文档进行正确的归类。

频繁项集挖掘：将一组项分组，并识别哪些个别项会经常一起出现。

12）ZooKeeper：Zookeeper是Google的Chubby一个开源的实现。它是一个针对大型分布式系统的可靠协调系统，提供的功能包括：配置维护、名字服务、 分布式同步、组服务等。ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

## 1.7 推荐系统框架图

![img](D:\software\Typora\imgs\clip_image018-1544403002408.png)

# 二 Hadoop运行环境搭建

## 2.1 虚拟机网络模式设置为NAT

![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image020.jpg) ![img](D:\software\Typora\imgs\clip_image022.jpg)

​       最后，重新启动系统。

​              [root@hadoop101 ~]# sync

[root@hadoop101 ~]# reboot

## 2.2 克隆虚拟机

1）克隆虚拟机

![img](D:\software\Typora\imgs\clip_image024.jpg)

 

![img](D:\software\Typora\imgs\clip_image026.jpg)

![img](D:\software\Typora\imgs\clip_image028.jpg)

![img](D:\software\Typora\imgs\clip_image030.jpg)

![img](D:\software\Typora\imgs\clip_image032.jpg)

![img](D:\software\Typora\imgs\clip_image034.jpg)

![img](D:\software\Typora\imgs\clip_image036.jpg)

2）启动虚拟机

## 2.3 修改为静态ip

1）在终端命令窗口中输入

[root@hadoop101 /]#vim /etc/udev/rules.d/70-persistent-net.rules

进入如下页面，删除eth0该行；将eth1修改为eth0，同时复制物理ip地址

![img](D:\software\Typora\imgs\clip_image038.jpg)

2）修改IP地址

[root@hadoop101 /]#vim /etc/sysconfig/network-scripts/ifcfg-eth0

需要修改的内容有5项：

IPADDR=192.168.1.101

GATEWAY=192.168.1.2

ONBOOT=yes

BOOTPROTO=static

DNS1=192.168.1.2

（1）修改前

![img](D:\software\Typora\imgs\clip_image040.jpg)

​       （2）修改后

![img](D:\software\Typora\imgs\clip_image041.png)

：wq  保存退出

3）执行service network restart

![img](D:\software\Typora\imgs\clip_image043-1544403002409.jpg)

4）如果报错，reboot，重启虚拟机

## 2.4 修改主机名

1）修改linux的hosts文件

（1）进入Linux系统查看本机的主机名。通过hostname命令查看

[root@hadoop ~]# hostname

hadoop100

（2）如果感觉此主机名不合适，我们可以进行修改。通过编辑/etc/sysconfig/network文件

\#vi /etc/sysconfig/network

 

文件中内容

NETWORKING=yes

NETWORKING_IPV6=no

HOSTNAME= hadoop101

注意：主机名称不要有“_”下划线

（3）打开此文件后，可以看到主机名。修改此主机名为我们想要修改的主机名hadoop101。

（4）保存退出。

（5）打开/etc/hosts

vim /etc/hosts

添加如下内容

192.168.1.100 hadoop100

192.168.1.101 hadoop101

192.168.1.102 hadoop102

192.168.1.103 hadoop103

192.168.1.104 hadoop104

192.168.1.105 hadoop105

192.168.1.106 hadoop106

192.168.1.107 hadoop107

192.168.1.108 hadoop108

192.168.1.109 hadoop109

192.168.1.110 hadoop110

（6）并重启设备，重启后，查看主机名，已经修改成功

2）修改window7的hosts文件

​       （1）进入C:\Windows\System32\drivers\etc路径

​       （2）打开hosts文件并添加如下内容

192.168.1.100 hadoop100

192.168.1.101 hadoop101

192.168.1.102 hadoop102

192.168.1.103 hadoop103

192.168.1.104 hadoop104

192.168.1.105 hadoop105

192.168.1.106 hadoop106

192.168.1.107 hadoop107

192.168.1.108 hadoop108

192.168.1.109 hadoop109

192.168.1.110 hadoop110

## 2.5 关闭防火墙

1）查看防火墙开机启动状态

chkconfig iptables --list  

2）关闭防火墙

chkconfig iptables off    

## 2.6 在opt目录下创建文件

1）创建atguigu用户

​       在root用户里面执行如下操作

   [root@hadoop101 opt]#   adduser atguigu   [root@hadoop101   opt]# passwd atguigu   更改用户 test 的密码 。   新的   密码：   无效的密码：   它没有包含足够的不同字符   无效的密码：   是回文   重新输入新的   密码：   passwd： 所有的身份验证令牌已经成功更新。   

2）设置atguigu用户具有root权限

修改 /etc/sudoers 文件，找到下面一行，在root下面添加一行，如下所示：

\## Allow root to run any commands anywhere

root    ALL=(ALL)     ALL

atguigu   ALL=(ALL)     ALL

修改完毕，现在可以用atguigu帐号登录，然后用命令 su - ，即可获得root权限进行操作。

3）在/opt目录下创建文件夹

（1）在root用户下创建module、software文件夹

mkdir module

mkdir software

​       （2）修改module、software文件夹的所有者

​       [root@hadoop101 opt]# chown atguigu module

[root@hadoop101 opt]# chown atguigu software

​              [root@hadoop101 opt]# ls -al

总用量 24

drwxr-xr-x.  6 root    root 4096 4月  24 09:07 .

dr-xr-xr-x. 23 root    root 4096 4月  24 08:52 ..

drwxr-xr-x.  4 atguigu root 4096 4月  23 16:26 module

drwxr-xr-x.  2 root    root 4096 3月  26 2015 rh

drwxr-xr-x.  2 atguigu root 4096 4月  23 16:25 software

## 2.7 安装jdk

1）卸载现有jdk

（1）查询是否安装java软件：

rpm -qa|grep java

（2）如果安装的版本低于1.7，卸载该jdk：

rpm -e 软件包

2）用filezilla工具将jdk、Hadoop-2.7.2.tar.gz导入到opt目录下面的software文件夹下面

![img](D:\software\Typora\imgs\clip_image045-1544403002409.jpg)

3）在linux系统下的opt目录中查看软件包是否导入成功。

[root@hadoop101opt]# cd software/

[root@hadoop101software]# ls

jdk-7u79-linux-x64.gz  hadoop-2.7.2.tar.gz    

4）解压jdk到/opt/module目录下

​       tar -zxf jdk-7u79-linux-x64.gz -C /opt/module/

5）配置jdk环境变量

​       （1）先获取jdk路径：

[root@hadoop101 jdk1.7.0_79]# pwd

/opt/module/jdk1.7.0_79     

​       （2）打开/etc/profile文件：

[root@hadoop101 jdk1.7.0_79]# vi /etc/profile

​              在profie文件末尾添加jdk路径：

​              \##JAVA_HOME

export JAVA_HOME=/opt/module/jdk1.7.0_79

export PATH=$PATH:$JAVA_HOME/bin

​       （3）保存后退出：

:wq

​       （4）让修改后的文件生效：

[root@hadoop101 jdk1.7.0_79]# source  /etc/profile

​       （5）重启（如果java –version可以用就不用重启）：    

[root@hadoop101 jdk1.7.0_79]# sync

​              [root@hadoop101 jdk1.7.0_79]# reboot

​       6）测试jdk安装成功

[root@hadoop101 jdk1.7.0_79]# java -version

java version "1.7.0_79"

## 2.8 安装Hadoop

1）进入到Hadoop安装包路径下：

[root@hadoop101 ~]# cd /opt/software/

2）解压安装文件到/opt/module下面

[root@hadoop101 software]# tar -zxf hadoop-2.7.2.tar.gz -C /opt/module/

3）查看是否解压成功

[root@hadoop101 software]# ls /opt/module/

hadoop-2.7.2  

4）配置hadoop中的hadoop-env.sh

   （1）Linux系统中获取jdk的安装路径：   [root@hadoop101   jdk1.7.0_79]# echo $JAVA_HOME   /opt/module/jdk1.7.0_79   （2）修改hadoop-env.sh文件中JAVA_HOME 路径：   export JAVA_HOME=/opt/module/jdk1.7.0_79   

5）将hadoop添加到环境变量

​       （1）获取hadoop安装路径：

[root@ hadoop101 hadoop-2.7.2]# pwd

/opt/module/hadoop-2.7.2

​       （2）打开/etc/profile文件：

root@ hadoop101 hadoop-2.7.2]# vi /etc/profile

​              在profie文件末尾添加jdk路径：（shitf+g）

\##HADOOP_HOME

export HADOOP_HOME=/opt/module/hadoop-2.7.2

export PATH=$PATH:$HADOOP_HOME/bin

export PATH=$PATH:$HADOOP_HOME/sbin                 

（3）保存后退出：

:wq

​       （4）让修改后的文件生效：

root@ hadoop101 hadoop-2.7.2]# source /etc/profile

​       （5）重启(如果hadoop命令不能用再重启)：  

root@ hadoop101 hadoop-2.7.2]# sync

​              root@ hadoop101 hadoop-2.7.2]# reboot

# 三 Hadoop运行模式 

1）官方网址

（1）官方网站：

<http://hadoop.apache.org/>

（2）各个版本归档库地址

<https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/>

（3）hadoop2.7.2版本详情介绍

<http://hadoop.apache.org/docs/r2.7.2/>

2）Hadoop运行模式

（1）本地模式（默认模式）：

不需要启用单独进程，直接可以运行，测试和开发时使用。

（2）伪分布式模式：

等同于完全分布式，只有一个节点。

（3）完全分布式模式：

多个节点一起运行。

## 3.1 本地文件运行Hadoop 案例

### 3.1.1 官方grep案例

1）创建在hadoop-2.7.2文件下面创建一个input文件夹

​     [atguigu@hadoop101 hadoop-2.7.2]$mkdir input

2）将hadoop的xml配置文件复制到input

​     [atguigu@hadoop101 hadoop-2.7.2]$cp etc/hadoop/*.xml input

3）执行share目录下的mapreduce程序

[atguigu@hadoop101 hadoop-2.7.2]$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep input output 'dfs[a-z.]+'

4）查看输出结果

[atguigu@hadoop101 hadoop-2.7.2]$ cat output/*

### 3.1.2 官方wordcount案例

1）创建在hadoop-2.7.2文件下面创建一个wcinput文件夹

[atguigu@hadoop101 hadoop-2.7.2]$mkdir wcinput

2）在wcinput文件下创建一个wc.input文件

[atguigu@hadoop101 hadoop-2.7.2]$cd wcinput

[atguigu@hadoop101 wcinput]$touch wc.input

3）编辑wc.input文件

   [atguigu@hadoop101 wcinput]$vim   wc.input   在文件中输入如下内容   hadoop yarn   hadoop mapreduce    atguigu   atguigu   保存退出：：wq   

4）回到hadoop目录/opt/module/hadoop-2.7.2

5）执行程序：

[atguigu@hadoop101 hadoop-2.7.2]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount wcinput wcoutput

6）查看结果：

[atguigu@hadoop101 hadoop-2.7.2]$cat wcoutput/part-r-00000

atguigu 2

hadoop  2

mapreduce       1

yarn    1

## 3.2 伪分布式运行Hadoop 案例

### 3.2.1 HDFS上运行MapReduce 程序

1）分析：

​       （1）准备1台客户机

​       （2）安装jdk

​       （3）配置环境变量

​       （4）安装hadoop

​       （5）配置环境变量

​       （6）配置集群

​       （7）启动、测试集群增、删、查

​       （8）在HDFS上执行wordcount案例

2）执行步骤

需要配置hadoop文件如下

（1）配置集群

​              （a）配置：hadoop-env.sh

​                     Linux系统中获取jdk的安装路径：

[root@ hadoop101 ~]# echo $JAVA_HOME

/opt/module/jdk1.7.0_79

​                     修改JAVA_HOME 路径：

   export JAVA_HOME=/opt/module/jdk1.7.0_79   

（b）配置：core-site.xml

   <!-- 指定HDFS中NameNode的地址 -->   <property>          <name>fs.defaultFS</name>       <value>hdfs://hadoop101:9000</value>   </property>       <!-- 指定hadoop运行时产生文件的存储目录 -->   <property>          <name>hadoop.tmp.dir</name>          <value>/opt/module/hadoop-2.7.2/data/tmp</value>   </property>   

（c）配置：hdfs-site.xml  

​          <!--   指定HDFS副本的数量 -->          <property>                 <name>dfs.replication</name>                 <value>1</value>          </property>   

（2）启动集群

（a）格式化namenode（第一次启动时格式化，以后就不要总格式化）

​                     bin/hdfs namenode -format

​              （b）启动namenode

​                     sbin/hadoop-daemon.sh start namenode

​              （c）启动datanode

​                     sbin/hadoop-daemon.sh start datanode

（3）查看集群

​              （a）查看是否启动成功

[root@hadoop101 ~]# jps

13586 NameNode

13668 DataNode

13786 Jps

​              （b）查看产生的log日志

当前目录：/opt/module/hadoop-2.7.2/logs

[root@hadoop101 logs]# ls

hadoop-root-datanode-hadoop.atguigu.com.log

hadoop-root-datanode-hadoop.atguigu.com.out

hadoop-root-namenode-hadoop.atguigu.com.log

hadoop-root-namenode-hadoop.atguigu.com.out

SecurityAuth-root.audit

[root@hadoop101 logs]# cat hadoop-root-datanode-hadoop.atguigu.com.log

​              （c）web端查看HDFS文件系统

​                     <http://192.168.1.101:50070/dfshealth.html#tab-overview>

​                     注意：如果不能查看，看如下帖子处理

<http://www.cnblogs.com/zlslch/p/6604189.html>

（4）操作集群

​              （a）在hdfs文件系统上**创建**一个input文件夹

[atguigu@hadoop101 hadoop-2.7.2]$ bin/hdfs dfs -mkdir -p /user/atguigu/mapreduce/wordcount/input

​              （b）将测试文件内容**上传**到文件系统上

bin/hdfs dfs -put wcinput/wc.input  /user/atguigu/mapreduce/wordcount/input/

​              （c）**查看**上传的文件是否正确

bin/hdfs dfs -ls  /user/atguigu/mapreduce/wordcount/input/

bin/hdfs dfs -cat  /user/atguigu/mapreduce/wordcount/input/wc.input

​              （d）在Hdfs上运行mapreduce程序

​                     bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/atguigu/mapreduce/wordcount/input/ /user/atguigu/mapreduce/wordcount/output

​              （e）查看输出结果

命令行查看：

bin/hdfs dfs -cat /user/atguigu/mapreduce/wordcount/output/*

浏览器查看

![img](D:\software\Typora\imgs\clip_image047.jpg)

​              （f）将测试文件内容**下载**到本地

hadoop fs -get /user/atguigu/mapreduce/wordcount/output/part-r-00000 ./wcoutput/

（g）**删除**输出结果

hdfs dfs -rmr /user/atguigu/mapreduce/wordcount/output

### 3.2.2 YARN上运行MapReduce 程序

1）分析：

​       （1）准备1台客户机

​       （2）安装jdk

​       （3）配置环境变量

​       （4）安装hadoop

​       （5）配置环境变量

​       （6）配置集群yarn上运行

​       （7）启动、测试集群增、删、查

​       （8）在yarn上执行wordcount案例

2）执行步骤

​       （1）配置集群

​              （a）配置yarn-env.sh

​       配置一下JAVA_HOME

   export   JAVA_HOME=/opt/module/jdk1.7.0_79   

（b）配置yarn-site.xml

   <!-- reducer获取数据的方式 -->   <property>    <name>yarn.nodemanager.aux-services</name>    <value>mapreduce_shuffle</value>   </property>       <!-- 指定YARN的ResourceManager的地址   -->   <property>   <name>yarn.resourcemanager.hostname</name>   <value>hadoop101</value>   </property>   

​              （c）配置：mapred-env.sh

​       配置一下JAVA_HOME

   export JAVA_HOME=/opt/module/jdk1.7.0_79   

​              （d）配置： (对mapred-site.xml.template重新命名为) mapred-site.xml

   <!-- 指定mr运行在yarn上 -->   <property>          <name>mapreduce.framework.name</name>          <value>yarn</value>   </property>   

​       （2）启动集群

（a）启动resourcemanager

sbin/yarn-daemon.sh start resourcemanager

（b）启动nodemanager

sbin/yarn-daemon.sh start nodemanager

​       （3）集群操作

（a）yarn的浏览器页面查看

http://192.168.1.101:8088/cluster

![img](D:\software\Typora\imgs\clip_image049.jpg)

​              （b）删除文件系统上的output文件

bin/hdfs dfs -rm -R /user/atguigu/mapreduce/wordcount/output

​              （c）执行mapreduce程序

​                     bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /user/atguigu/mapreduce/wordcount/input  /user/atguigu/mapreduce/wordcount/output

​              （d）查看运行结果

bin/hdfs dfs -cat /user/atguigu/mapreduce/wordcount/output/*

![img](D:\software\Typora\imgs\clip_image051.jpg)

### 3.2.3 修改本地临时文件存储目录

1）停止进程

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop resourcemanager

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/hadoop-daemon.sh stop datanode

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/hadoop-daemon.sh stop namenode

2）修改hadoop.tmp.dir

​       [core-site.xml]

​          <!--   指定hadoop运行时产生文件的存储目录 -->          <property>                 <name>hadoop.tmp.dir</name>                 <value>/opt/module/hadoop-2.7.2/data/tmp</value>          </property>   

3）格式化NameNode

​       [atguigu@hadoop101 hadoop-2.7.2]$ hadoop namenode -format

4）启动所有进程

​       [atguigu@hadoop101 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start namenode

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start datanode

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh start resourcemanager

[atguigu@hadoop101 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager

​       5）查看/opt/module/hadoop-2.7.2/data/tmp这个目录下的内容。

### 3.2.4 Hadoop配置文件说明

Hadoop配置文件分两类：默认配置文件和自定义配置文件，只有用户想修改某一默认配置值时，才需要修改自定义配置文件，更改相应属性值。

（1）默认配置文件：存放在hadoop相应的jar包中

[core-default.xml]

​                     hadoop-common-2.7.2.jar/ core-default.xml

​              [hdfs-default.xml]

hadoop-hdfs-2.7.2.jar/ hdfs-default.xml

​              [yarn-default.xml]

hadoop-yarn-common-2.7.2.jar/ yarn-default.xml

​              [core-default.xml]

hadoop-mapreduce-client-core-2.7.2.jar/ core-default.xml

​       （2）自定义配置文件：存放在$HADOOP_HOME/etc/hadoop

​              core-site.xml

​              hdfs-site.xml

​              yarn-site.xml

​              mapred-site.xml

### 3.2.5 历史服务配置启动查看

​       1）配置mapred-site.xml

   <property>   <name>mapreduce.jobhistory.address</name>   <value>hadoop101:10020</value>   </property>   <property>         <name>mapreduce.jobhistory.webapp.address</name>         <value>hadoop101:19888</value>   </property>   

​       2）查看启动历史服务器文件目录：

[root@hadoop101 hadoop-2.7.2]# ls sbin/ |grep mr

mr-jobhistory-daemon.sh

​       3）启动历史服务器

sbin/mr-jobhistory-daemon.sh start historyserver

​       4）查看历史服务器是否启动

​              jps

​       5）查看jobhistory

<http://192.168.1.101:19888/jobhistory>

### 3.2.6 日志的聚集

​       日志聚集概念：应用运行完成以后，将日志信息上传到HDFS系统上

​       开启日志聚集功能步骤：

（1）配置yarn-site.xml

   <!-- 日志聚集功能使能 -->   <property>   <name>yarn.log-aggregation-enable</name>   <value>true</value>   </property>   <!-- 日志保留时间设置7天 -->   <property>   <name>yarn.log-aggregation.retain-seconds</name>   <value>604800</value>   </property>   

（2）关闭nodemanager 、resourcemanager和historymanager

​       sbin/yarn-daemon.sh stop resourcemanager

sbin/yarn-daemon.sh stop nodemanager

sbin/mr-jobhistory-daemon.sh stop historyserver

（3）启动nodemanager 、resourcemanager和historymanager

sbin/yarn-daemon.sh start resourcemanager

sbin/yarn-daemon.sh start nodemanager

sbin/mr-jobhistory-daemon.sh start historyserver

​              （4）删除hdfs上已经存在的hdfs文件

bin/hdfs dfs -rm -R /user/atguigu/mapreduce/wordcount/output

​              （5）执行wordcount程序

hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.0.jar wordcount /user/atguigu/mapreduce/wordcount/input /user/atguigu/mapreduce/wordcount/output

​              （6）查看日志

<http://192.168.1.101:19888/jobhistory>

![img](D:\software\Typora\imgs\clip_image053.jpg)

![img](D:\software\Typora\imgs\clip_image055.jpg)

![img](D:\software\Typora\imgs\clip_image057.jpg)

## 3.3 完全分布式部署Hadoop

分析：

​       1）准备3台客户机（关闭防火墙、静态ip、主机名称）

​       2）安装jdk

​       3）配置环境变量

​       4）安装hadoop

​       5）配置环境变量

​       6）安装ssh

​       7）配置集群

​       8）启动测试集群

### 3.3.1 虚拟机准备

详见2.2-2.3章。

### 3.3.2 主机名设置

详见2.4章。

### 3.3.3 scp

1）scp可以实现服务器与服务器之间的数据拷贝。

2）案例实操

（1）将hadoop101中/opt/module和/opt/software文件拷贝到hadoop102、hadoop103和hadoop104上。

[root@hadoop101 /]# scp -r /opt/module/  root@hadoop102:/opt

[root@hadoop101 /]# scp -r /opt/software/  root@hadoop102:/opt

[root@hadoop101 /]# scp -r /opt/module/  root@hadoop103:/opt

[root@hadoop101 /]# scp -r /opt/software/  root@hadoop103:/opt

[root@hadoop101 /]# scp -r /opt/module/  root@hadoop104:/opt

[root@hadoop101 /]# scp -r /opt/software/  root@hadoop105:/opt

（2）将192.168.1.102服务器上的文件拷贝到当前用户下。

[root@hadoop101 opt]# scp  root@hadoop102:/etc/profile  /opt/tmp/

​       （3）实现两台远程机器之间的文件传输（hadoop103主机文件拷贝到hadoop104主机上）

​             [atguigu@hadoop102 test]$ scp atguigu@hadoop103:/opt/test/haha atguigu@hadoop104:/opt/test/

### 3.3.4 SSH无密码登录

1）配置ssh

（1）基本语法

ssh 另一台电脑的ip地址

（2）ssh连接时出现Host key verification failed的解决方法

[root@hadoop2 opt]# ssh 192.168.1.103

The authenticity of host '192.168.1.103 (192.168.1.103)' can't be established.

RSA key fingerprint is cf:1e:de:d7:d0:4c:2d:98:60:b4:fd:ae:b1:2d:ad:06.

Are you sure you want to continue connecting (yes/no)? 

Host key verification failed.

（3）解决方案如下：直接输入yes

2）无密钥配置

（1）进入到我的home目录

​              cd  ~/.ssh

（2）生成公钥和私钥：

ssh-keygen -t rsa 

然后敲（三个回车），就会生成两个文件id_rsa（私钥）、id_rsa.pub（公钥）

（3）将公钥拷贝到要免密登录的目标机器上

ssh-copy-id 192.168.1.102

![img](D:\software\Typora\imgs\clip_image059.png)3）.ssh文件夹下的文件功能解释

​       （1）~/.ssh/known_hosts      ：记录ssh访问过计算机的公钥(public key)

​       （2）id_rsa    ：生成的私钥

​       （3）id_rsa.pub     ：生成的公钥

​       （4）authorized_keys    ：存放授权过得无秘登录服务器公钥

### 3.3.5 rsync

rsync远程同步工具，主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。

（1）查看rsync使用说明

man rsync | more

​       （2）基本语法

rsync -rvl     $pdir/$fname         $user@hadoop$host:$pdir

​              命令 命令参数 要拷贝的文件路径/名称   目的用户@主机:目的路径

​              选项

-r 递归

-v 显示复制过程

-l 拷贝符号连接

​       （3）案例实操

​              把本机/opt/tmp目录同步到hadoop103服务器的root用户下的/opt/tmp目录

​              rsync -rvl /opt/tmp/*  root@hadoop103:/op t/tmp

### 3.3.6 编写集群分发脚本xsync

1）需求分析：循环复制文件到所有节点的相同目录下。

​       （1）原始拷贝：

rsync  -rvl     /opt/module              root@hadoop103:/opt/

​       （2）期望脚本：

xsync 要同步的文件名称

​       （3）在/usr/local/bin这个目录下存放的脚本，可以在系统任何地方直接执行，需要制定路径。

2）案例实操：

（1）在/usr/local/bin目录下创建xsync文件，文件内容如下：

   #!/bin/bash   #1 获取输入参数个数，如果没有参数，直接退出   pcount=$#   if((pcount==0));   then   echo no args;   exit;   fi       #2 获取文件名称   p1=$1   fname=`basename   $p1`   echo   fname=$fname       #3 获取上级目录到绝对路径   pdir=`cd -P   $(dirname $p1); pwd`   echo pdir=$pdir       #4 获取当前用户名称   user=`whoami`       #5 循环   for((host=103;   host<105; host++)); do           #echo $pdir/$fname   $user@hadoop$host:$pdir           echo --------------- hadoop$host ----------------           rsync -rvl $pdir/$fname   $user@hadoop$host:$pdir   done   

（2）修改脚本 xsync 具有执行权限

​             [root@hadoop102 bin]# chmod a+x xsync

（3）调用脚本形式：xsync 文件名称

### 3.3.7 编写分发脚本xcall

1）需求分析：在所有主机上同时执行相同的命令

xcall +命令           

2）具体实现

（1）在/usr/local/bin目录下创建xcall文件，文件内容如下：

   #!/bin/bash   pcount=$#   if((pcount==0));then           echo no args;           exit;   fi       echo -------------localhost----------   $@   for((host=101;   host<=108; host++)); do           echo ----------hadoop$host---------           ssh hadoop$host $@   done   

（2）修改脚本 xcall 具有执行权限

​             [root@hadoop102 bin]# chmod a+x xcall

（3）调用脚本形式： xcall 操作命令

[root@hadoop102 ~]# xcall rm -rf /opt/tmp/profile 

### 3.3.8 配置集群

1）集群部署规划

|      | Hadoop102           | hadoop103                     | hadoop104                    |
| ---- | ------------------- | ----------------------------- | ---------------------------- |
| HDFS | NameNode   DataNode | DataNode                      | SecondaryNameNode   DataNode |
| YARN | NodeManager         | ResourceManager   NodeManager | NodeManager                  |

2）配置文件

​       （1）core-site.xml

   <!-- 指定HDFS中NameNode的地址 -->          <property>                 <name>fs.defaultFS</name>             <value>hdfs://hadoop102:9000</value>          </property>          <!-- 指定hadoop运行时产生文件的存储目录 -->          <property>                 <name>hadoop.tmp.dir</name>                 <value>/opt/module/hadoop-2.7.2/data/tmp</value>          </property>   

​       （2）Hdfs

​              hadoop-env.sh

   export   JAVA_HOME=/opt/module/jdk1.7.0_79   

​              hdfs-site.xml

   <configuration>               <property>                 <name>dfs.replication</name>                 <value>3</value>          </property>          <property>             <name>dfs.namenode.secondary.http-address</name>             <value>hadoop104:50090</value>         </property>   </configuration>   

​              slaves

   hadoop102   hadoop103   hadoop104   

​       （3）yarn

​              yarn-env.sh

   export JAVA_HOME=/opt/module/jdk1.7.0_79   

​              yarn-site.xml

   <configuration>   <!-- Site specific YARN configuration properties   -->   <!-- reducer获取数据的方式 -->       <property>             <name>yarn.nodemanager.aux-services</name>             <value>mapreduce_shuffle</value>         </property>       <!-- 指定YARN的ResourceManager的地址   -->          <property>                 <name>yarn.resourcemanager.hostname</name>                 <value>hadoop103</value>          </property>          </configuration>   

​       （4）mapreduce

​              mapred-env.sh

   export JAVA_HOME=/opt/module/jdk1.7.0_79   

​              mapred-site.xml

   <configuration>   <!-- 指定mr运行在yarn上   -->          <property>                 <name>mapreduce.framework.name</name>                 <value>yarn</value>          </property>          </configuration>   

3）在集群上分发以上所有文件

cd /opt/module/hadoop-2.7.2/etc/hadoop

xsync /opt/module/hadoop-2.7.2/etc/hadoop/core-site.xml

xsync /opt/module/hadoop-2.7.2/etc/hadoop/yarn-site.xml

xsync /opt/module/hadoop-2.7.2/etc/hadoop/slaves

4）查看文件分发情况

​       xcall cat /opt/module/hadoop-2.7.2/etc/hadoop/slaves

### 3.3.9 集群启动及测试

1）启动集群

​       （0）如果集群是第一次启动，需要格式化namenode

​              [root@hadoop102 hadoop-2.7.2]# bin/hdfs namenode -format

（1）启动HDFS：

[root@hadoop102 hadoop-2.7.2]# sbin/start-dfs.sh

[root@hadoop102 hadoop-2.7.2]# jps

4166 NameNode

4482 Jps

4263 DataNode

 

[root@hadoop103 桌面]# jps

3218 DataNode

3288 Jps

 

[root@hadoop104 桌面]# jps

3221 DataNode

3283 SecondaryNameNode

3364 Jps

（2）启动yarn

sbin/start-yarn.sh

注意：Namenode和ResourceManger如果不是同一台机器，不能在NameNode上启动 yarn，应该在ResouceManager所在的机器上启动yarn。

2）集群基本测试

（1）上传文件到集群

​       上传小文件

​       bin/hdfs dfs -mkdir -p /user/atguigu/tmp/conf

​       bin/hdfs dfs -put etc/hadoop/*-site.xml /user/atguigu/tmp/conf

​       上传大文件

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -put /opt/software/hadoop-2.7.2.tar.gz  /user/atguigu/input

（2）上传文件后查看文件存放在什么位置

​       文件存储路径

​       [atguigu@hadoop102 subdir0]$ pwd

/opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/BP-938951106-192.168.10.107-1495462844069/current/finalized/subdir0/subdir0

​       查看文件内容

[atguigu@hadoop102 subdir0]$ cat blk_1073741825

hadoop

atguigu

atguigu

（3）拼接

-rw-rw-r--. 1 atguigu atguigu 134217728 5月  23 16:01 blk_1073741836

-rw-rw-r--. 1 atguigu atguigu   1048583 5月  23 16:01 blk_1073741836_1012.meta

-rw-rw-r--. 1 atguigu atguigu  63439959 5月  23 16:01 blk_1073741837

-rw-rw-r--. 1 atguigu atguigu    495635 5月  23 16:01 blk_1073741837_1013.meta

[atguigu@hadoop102 subdir0]$ cat blk_1073741836>>tmp.file

[atguigu@hadoop102 subdir0]$ cat blk_1073741837>>tmp.file

[atguigu@hadoop102 subdir0]$ tar -zxvf tmp.file

（4）下载

[atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -get /user/atguigu/input/hadoop-2.7.2.tar.gz

3）性能测试集群

​       写海量数据

​       读海量数据

### 3.3.10 Hadoop启动停止方式

1）各个服务组件逐一启动

​       （1）分别启动hdfs组件

​              hadoop-daemon.sh  start|stop  namenode|datanode|secondarynamenode

​       （2）启动yarn

​              yarn-daemon.sh  start|stop  resourcemanager|nodemanager

2）各个模块分开启动（配置ssh是前提）常用

​       （1）整体启动/停止hdfs

​              start-dfs.sh

​              stop-dfs.sh

​       （2）整体启动/停止yarn

​              start-yarn.sh

​              stop-yarn.sh

3）全部启动（不建议使用）

​       start-all.sh

​       stop-all.sh

### 3.3.11 集群时间同步

时间同步的方式：找一个机器，作为时间服务器，所有的机器与这台集群时间进行定时的同步，比如，每日十分钟，同步一次时间。

配置时间同步：

1）时间服务器配置

（1）检查ntp是否安装

[root@hadoop102 桌面]# rpm -qa|grep ntp

ntp-4.2.6p5-10.el6.centos.x86_64

fontpackages-filesystem-1.41-1.1.el6.noarch

ntpdate-4.2.6p5-10.el6.centos.x86_64

（2）修改ntp配置文件

vi /etc/ntp.conf

修改内容如下

a）修改1

\#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap为

restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap

​              b）修改2

server 0.centos.pool.ntp.org iburst

server 1.centos.pool.ntp.org iburst

server 2.centos.pool.ntp.org iburst

server 3.centos.pool.ntp.org iburst为

\#server 0.centos.pool.ntp.org iburst

\#server 1.centos.pool.ntp.org iburst

\#server 2.centos.pool.ntp.org iburst

\#server 3.centos.pool.ntp.org iburst

​              c）添加3

server 127.127.1.0

fudge 127.127.1.0 stratum 10

3）修改/etc/sysconfig/ntpd 文件

vim /etc/sysconfig/ntpd

增加内容如下

SYNC_HWCLOCK=yes

​       4）重新启动ntpd

[root@hadoop102 桌面]# service ntpd status

ntpd 已停

[root@hadoop102 桌面]# service ntpd start

正在启动 ntpd：                                            [确定]

​       5）执行：

​              chkconfig ntpd on

2）其他机器配置（必须root用户）

​       （1）在其他机器配置10分钟与时间服务器同步一次

​              [root@hadoop103 hadoop-2.7.2]# crontab -e

​              编写脚本

​              */10 * * * * /usr/sbin/ntpdate hadoop102

​       （2）修改任意机器时间

​              date -s "2015-9-11"

​       （3）十分钟后查看机器是否与时间服务器同步

​              date

### 3.3.12 配置集群常见问题

1）防火墙没关闭、或者没有启动yarn

*INFO client.RMProxy: Connecting to ResourceManager at hadoop108/192.168.10.108:8032*

2）主机名称配置错误

3）ip地址配置错误

4）ssh没有配置好

5）root用户和atguigu两个用户启动集群不统一

6）配置文件修改不细心

7）未编译源码

*Unable to load native-hadoop library for your platform... using builtin-java classes where applicable*

*17/05/22 15:38:58 INFO client.RMProxy: Connecting to ResourceManager at hadoop108/192.168.10.108:8032*

8）datanode不被namenode识别问题

Namenode在format初始化的时候会形成两个标识，blockPoolId和clusterId。新的datanode加入时，会获取这两个标识作为自己工作目录中的标识。

一旦namenode重新format后，namenode的身份标识已变，而datanode如果依然持有原来的id，就不会被namenode识别。

解决办法，删除datanode节点中的数据后，再次重新格式化namenode。

9）不识别主机名称

   java.net.UnknownHostException:   hadoop102: hadoop102           at   java.net.InetAddress.getLocalHost(InetAddress.java:1475)           at   org.apache.hadoop.mapreduce.JobSubmitter.submitJobInternal(JobSubmitter.java:146)           at   org.apache.hadoop.mapreduce.Job$10.run(Job.java:1290)           at   org.apache.hadoop.mapreduce.Job$10.run(Job.java:1287)           at   java.security.AccessController.doPrivileged(Native Method)           at   javax.security.auth.Subject.doAs(Subject.java:415)   

解决办法：

（1）在/etc/hosts文件中添加192.168.1.102 hadoop102

​       （2）主机名称不要起hadoop  hadoop000等特殊名称

10）datanode和namenode进程同时只能工作一个。

![img](D:\software\Typora\imgs\clip_image061.png)

11）执行命令 不生效，粘贴word中命令时，遇到-和长–没区分开。导致命令失效

解决办法：尽量不要粘贴word中代码。

12）jps发现进程已经没有，但是重新启动集群，提示进程已经开启。原因是在linux的根目录下/tmp目录中存在启动的进程临时文件，将集群相关进程删除掉，再重新启动集群。

# 四 Hadoop编译源码

## 4.1 前期准备工作

1）CentOS联网 

[root@hadoop101 桌面]# vi /etc/sysconfig/network-scripts/ifcfg-eth0 

   DEVICE=eth0   HWADDR=00:0c:29:ca:6e:ec   TYPE=Ethernet   UUID=9e008bf7-44f6-4e72-8ead-71b8ea7a9b5b   ONBOOT=yes   NM_CONTROLLED=yes   BOOTPROTO=dhcp   

[root@hadoop101 桌面]# service network restart

注意：采用root角色编译，减少文件夹权限出现问题

2）jar包准备(hadoop源码、JDK7 、 maven、 ant 、protobuf)

（1）hadoop-2.7.2-src.tar.gz

（2）jdk-7u79-linux-x64.gz

（3）apache-ant-1.9.9-bin.tar.gz

（4）apache-maven-3.0.5-bin.tar.gz

（5）protobuf-2.5.0.tar.gz

## 4.2 jar包安装

0）注意：所有操作必须在root用户下完成

1）JDK解压、配置环境变量 JAVA_HOME和PATH，验证[java](http://lib.csdn.net/base/javase)-version(如下都需要验证是否配置成功)

[root@hadoop101 software] # tar -zxf jdk-7u79-linux-x64.gz -C /opt/module/

[root@hadoop101 software]# vi /etc/profile

   #JAVA_HOME   export JAVA_HOME=/opt/module/jdk1.7.0_79   export PATH=$PATH:$JAVA_HOME/bin   

[root@hadoop101 software]#source /etc/profile

验证命令：java -version

2）Maven解压、配置  MAVEN_HOME和PATH。

[root@hadoop101 software]# tar -zxvf apache-maven-3.0.5-bin.tar.gz -C /opt/module/

[root@hadoop101 apache-maven-3.0.5]#  vi /etc/profile

   #MAVEN_HOME   export   MAVEN_HOME=/opt/module/apache-maven-3.0.5   export PATH=$PATH:$MAVEN_HOME/bin   

[root@hadoop101 software]#source /etc/profile

验证命令：mvn -version

3）ant解压、配置  ANT _HOME和PATH。

[root@hadoop101 software]# tar -zxvf apache-ant-1.9.9-bin.tar.gz -C /opt/module/

[root@hadoop101 apache-ant-1.9.9]# vi /etc/profile

   #ANT_HOME   export   ANT_HOME=/opt/module/apache-ant-1.9.9   export PATH=$PATH:$ANT_HOME/bin   

[root@hadoop101 software]#source /etc/profile

验证命令：ant -version

4）安装  glibc-headers 和  g++  命令如下: 

[root@hadoop101 apache-ant-1.9.9]# yum install glibc-headers

[root@hadoop101 apache-ant-1.9.9]# yum install gcc-c++

5）安装make和cmake

[root@hadoop101 apache-ant-1.9.9]# yum install make

[root@hadoop101 apache-ant-1.9.9]# yum install cmake

6）解压protobuf ，进入到解压后protobuf主目录，/opt/module/protobuf-2.5.0

然后相继执行命令：

[root@hadoop101 software]# tar -zxvf protobuf-2.5.0.tar.gz -C /opt/module/

[root@hadoop101 opt]# cd /opt/module/protobuf-2.5.0/

 

[root@hadoop101 protobuf-2.5.0]#./configure 

[root@hadoop101 protobuf-2.5.0]# make 

[root@hadoop101 protobuf-2.5.0]# make check 

[root@hadoop101 protobuf-2.5.0]# make install 

[root@hadoop101 protobuf-2.5.0]# ldconfig 

 

[root@hadoop101 hadoop-dist]# vi /etc/profile

   #LD_LIBRARY_PATH   export   LD_LIBRARY_PATH=/opt/module/protobuf-2.5.0   export PATH=$PATH:$LD_LIBRARY_PATH   

[root@hadoop101 software]#source /etc/profile

验证命令：protoc --version

7）安装openssl库

[root@hadoop101 software]#yum install openssl-devel

8）安装 ncurses-devel库：

[root@hadoop101 software]#yum install ncurses-devel

到此，编译工具安装基本完成。

## 4.3 编译源码

1）解压源码到/opt/tools目录

[root@hadoop101 software]# tar -zxvf hadoop-2.7.2-src.tar.gz -C /opt/

2）进入到hadoop源码主目录

[root@hadoop101 hadoop-2.7.2-src]# pwd

/opt/hadoop-2.7.2-src

3）通过maven执行编译命令

[root@hadoop101 hadoop-2.7.2-src]#mvn package -Pdist,native -DskipTests -Dtar

等待时间30分钟左右，最终成功是全部SUCCESS。

![img](D:\software\Typora\imgs\clip_image063.jpg)

4）成功的64位hadoop包在/opt/hadoop-2.7.2-src/hadoop-dist/target下。

[root@hadoop101 target]# pwd

/opt/hadoop-2.7.2-src/hadoop-dist/target

## 4.4 常见的问题及解决方案

1）MAVEN install时候JVM内存溢出

处理方式：在环境配置文件和maven的执行文件均可调整MAVEN_OPT的heap大小。（详情查阅MAVEN 编译 JVM调优问题，如：http://outofmemory.cn/code-snippet/12652/maven-outofmemoryerror-method）

2）编译期间maven报错。可能网络阻塞问题导致依赖库下载不完整导致，多次执行命令（一次通过比较难）：

[root@hadoop101 hadoop-2.7.2-src]#mvn package -Pdist,native -DskipTests -Dtar

3）报ant、protobuf等错误，插件下载未完整或者插件版本问题，最开始链接有较多特殊情况，同时推荐

2.7.0版本的问题汇总帖子   http://www.tuicool.com/articles/IBn63qf

 