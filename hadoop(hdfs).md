尚硅谷大数据技术之Hadoop

（HDFS文件系统）

(作者：大海哥)

 

官网：[www.atguigu.com](http://www.atguigu.com)

版本：V1.1

# 一 HDFS概念

## 1.1 概念

**HDFS****，它是一个文件系统**，用于存储文件，通过目录树来定位文件；**其次，它是分布式的**，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

HDFS的设计适合一次写入，多次读出的场景，且不支持文件的修改。适合用来做数据分析，并不适合用来做网盘应用。

## 1.2 组成

1）HDFS集群包括，NameNode和DataNode以及Secondary Namenode。

2）NameNode负责管理整个文件系统的元数据，以及每一个路径（文件）所对应的数据块信息。

3）DataNode 负责管理用户的文件数据块，每一个数据块都可以在多个datanode上存储多个副本。

4）Secondary NameNode用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

## 1.3 HDFS 文件块大小

HDFS中的文件在物理上是分块存储（block），块的大小可以通过配置参数( dfs.blocksize)来规定，默认大小在hadoop2.x版本中是128M，老版本中是64M

HDFS的块比磁盘的块大，其目的是为了最小化寻址开销。如果块设置得足够大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。因而，传输一个由多个块组成的文件的时间取决于磁盘传输速率。

如果寻址时间约为10ms，而传输速率为100MB/s，为了使寻址时间仅占传输时间的1%，我们要将块大小设置约为100MB。默认的块大小实际为64MB，但是很多情况下HDFS使用128MB的块设置。

块的大小：10ms*100*100M/s = 100M

![img](D:\software\Typora\imgs\clip_image002.png)

# 二 HFDS命令行操作

1）基本语法

bin/hadoop fs 具体命令

2）参数大全

​       bin/hadoop fs

   [-appendToFile   <localsrc> ... <dst>]           [-cat [-ignoreCrc]   <src> ...]           [-checksum   <src> ...]           [-chgrp [-R] GROUP   PATH...]           [-chmod [-R] <MODE[,MODE]... |   OCTALMODE> PATH...]           [-chown [-R]   [OWNER][:[GROUP]] PATH...]           [-copyFromLocal   [-f] [-p] <localsrc> ... <dst>]           [-copyToLocal [-p]   [-ignoreCrc] [-crc] <src> ... <localdst>]           [-count [-q]   <path> ...]           [-cp [-f] [-p]   <src> ... <dst>]           [-createSnapshot   <snapshotDir> [<snapshotName>]]           [-deleteSnapshot   <snapshotDir> <snapshotName>]           [-df [-h]   [<path> ...]]           [-du [-s] [-h]   <path> ...]           [-expunge]           [-get [-p]   [-ignoreCrc] [-crc] <src> ... <localdst>]           [-getfacl [-R]   <path>]           [-getmerge [-nl]   <src> <localdst>]           [-help [cmd ...]]           [-ls [-d] [-h]   [-R] [<path> ...]]           [-mkdir [-p]   <path> ...]           [-moveFromLocal   <localsrc> ... <dst>]           [-moveToLocal   <src> <localdst>]           [-mv <src>   ... <dst>]           [-put [-f] [-p]   <localsrc> ... <dst>]           [-renameSnapshot   <snapshotDir> <oldName> <newName>]           [-rm [-f] [-r|-R]   [-skipTrash] <src> ...]           [-rmdir   [--ignore-fail-on-non-empty] <dir> ...]           [-setfacl [-R]   [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec>   <path>]]           [-setrep [-R] [-w]   <rep> <path> ...]           [-stat [format]   <path> ...]           [-tail [-f]   <file>]           [-test -[defsz]   <path>]           [-text   [-ignoreCrc] <src> ...]           [-touchz   <path> ...]           [-usage [cmd ...]]   

3）常用命令实操

（1）-help：输出这个命令参数

​              bin/hdfs dfs -help rm

​       （2）-ls: 显示目录信息

hadoop fs -ls /

（3）-mkdir：在hdfs上创建目录

hadoop fs  -mkdir  -p  /aaa/bbb/cc/dd

（4）-moveFromLocal从本地剪切粘贴到hdfs

hadoop  fs  - moveFromLocal  /home/hadoop/a.txt  /aaa/bbb/cc/dd

（5）-moveToLocal：从hdfs剪切粘贴到本地

hadoop  fs  - moveToLocal   /aaa/bbb/cc/dd  /home/hadoop/a.txt

（6）--appendToFile  ：追加一个文件到已经存在的文件末尾

hadoop  fs  -appendToFile  ./hello.txt  /hello.txt

（7）-cat ：显示文件内容

（8）-tail：显示一个文件的末尾

hadoop  fs  -tail  /weblog/access_log.1

（9）-text：以字符形式打印一个文件的内容

hadoop  fs  -text  /weblog/access_log.1

（10）-chgrp 、-chmod、-chown：linux文件系统中的用法一样，修改文件所属权限

hadoop  fs  -chmod  666  /hello.txt

hadoop  fs  -chown  someuser:somegrp   /hello.txt

（11）-copyFromLocal：从本地文件系统中拷贝文件到hdfs路径去

hadoop  fs  -copyFromLocal  ./jdk.tar.gz  /aaa/

（12）-copyToLocal：从hdfs拷贝到本地

hadoop fs -copyToLocal /aaa/jdk.tar.gz

（13）-cp ：从hdfs的一个路径拷贝到hdfs的另一个路径

hadoop  fs  -cp  /aaa/jdk.tar.gz  /bbb/jdk.tar.gz.2

（14）-mv：在hdfs目录中移动文件

hadoop  fs  -mv  /aaa/jdk.tar.gz  /

（15）-get：等同于copyToLocal，就是从hdfs下载文件到本地

hadoop fs -get  /aaa/jdk.tar.gz

（16）-getmerge  ：合并下载多个文件，比如hdfs的目录 /aaa/下有多个文件:log.1, *log.2,log.3,...*

hadoop fs -getmerge /aaa/log.* ./log.sum

（17）-put：等同于copyFromLocal

hadoop  fs  -put  /aaa/jdk.tar.gz  /bbb/jdk.tar.gz.2

（18）-rm：删除文件或文件夹

hadoop fs -rm -r /aaa/bbb/

（19）-rmdir：删除空目录

hadoop  fs  -rmdir   /aaa/bbb/ccc

（20）-df ：统计文件系统的可用空间信息

hadoop  fs  -df  -h  /

（21）-du统计文件夹的大小信息

hadoop  fs  -du  -s  -h /aaa/*

（22）-count：统计一个指定目录下的文件节点数量

hadoop fs -count /aaa/

（23）-setrep：设置hdfs中文件的副本数量

hadoop fs -setrep 3 /aaa/jdk.tar.gz

![img](D:\software\Typora\imgs\clip_image004-1544403202080.jpg)

这里设置的副本数只是记录在namenode的元数据中，是否真的会有这么多副本，还得看datanode的数量。因为目前只有3台设备，最多也就3个副本，只有节点数的增加到10台时，副本数才能达到10。

# 三 HDFS客户端操作

## 3.1 eclipse环境准备

### 3.1.1 jar包准备

1）解压hadoop-2.7.2.tar.gz到非中文目录

2）进入share文件夹，查找所有jar包，并把jar包拷贝到_lib文件夹下

3）在全部jar包中查找.source.jar，并剪切到_source文件夹。

4）在全部jar包中查找tests.jar，并剪切到_test文件夹。

### 3.1.2 eclipse准备

1）配置HADOOP_HOME环境变量

2）采用hadoop编译后的bin 、lib两个文件夹（如果不生效，重新启动eclipse）

3）创建第一个java工程

   **public** **class** HdfsClientDemo1 {          **public** **static** **void**   main(String[] args) **throws** Exception {                 // 1 获取文件系统                 Configuration configuration = **new**   Configuration();                 // 配置在集群上运行                 configuration.set("fs.defaultFS",   "hdfs://hadoop102:9000");                 FileSystem fileSystem =   FileSystem.*get*(configuration);                                  // 直接配置访问集群的路径和访问集群的用户名称   //            FileSystem   fileSystem = FileSystem.get(new URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                  // 2 把本地文件上传到文件系统中                 fileSystem.copyFromLocalFile(**new**   Path("f:/hello.txt"), **new**   Path("/hello1.copy.txt"));                                  // 3 关闭资源                 fileSystem.close();                 System.*out*.println("over");          }   }   

4）执行程序

运行时需要配置用户名称

![img](D:\software\Typora\imgs\clip_image006.jpg)

客户端去操作hdfs时，是有一个用户身份的。默认情况下，hdfs客户端api会从jvm中获取一个参数来作为自己的用户身份：-DHADOOP_USER_NAME=atguigu，atguigu为用户名称。

## 3.2 通过API操作HDFS

### 3.2.1 HDFS获取文件系统

1）详细代码

​          @Test          **public**   **void** initHDFS() **throws** Exception{                 //   1 创建配置信息对象                 //   new Configuration();的时候，它就会去加载jar包中的hdfs-default.xml                 //   然后再加载classpath下的hdfs-site.xml                 Configuration   configuration = **new** Configuration();                                  //   2 设置参数                  //   参数优先级： 1、客户端代码中设置的值  2、classpath下的用户自定义配置文件 3、然后是服务器的默认配置   //            configuration.set("fs.defaultFS",   "hdfs://hadoop102:9000");                 configuration.set("dfs.replication",   "3");                                  //   3 获取文件系统                 FileSystem   fs = FileSystem.*get*(configuration);                                  //   4 打印文件系统                 System.*out*.println(fs.toString());          }   

2）将core-site.xml拷贝到项目的根目录下

   <?xml version=*"1.0"*   encoding=*"UTF-8"*?>   <?xml-stylesheet type=*"text/xsl"*   href=*"configuration.xsl"*?>       <configuration>   <!-- 指定HDFS中NameNode的地址 -->          <property>                 <name>fs.defaultFS</name>             <value>hdfs://hadoop102:9000</value>          </property>              <!--   指定hadoop运行时产生文件的存储目录 -->          <property>                 <name>hadoop.tmp.dir</name>                 <value>/opt/module/hadoop-2.7.2/data/tmp</value>          </property>   </configuration>   

### 3.2.2 HDFS文件上传

​          @Test          **public**   **void** putFileToHDFS() **throws** Exception{                 //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                                  FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                  //   2 创建要上传文件所在的本地路径                 Path   src = **new** Path("e:/hello.txt");                                  //   3 创建要上传到hdfs的目标路径                 Path   dst = **new** Path("hdfs://hadoop102:9000/user/atguigu/hello.txt");                                  //   4 拷贝文件                 fs.copyFromLocalFile(src,   dst);                 fs.close();    }   

### 3.2.3 HDFS文件下载

   @Test   **public** **void** getFileFromHDFS() **throws**   Exception{                           // 1 创建配置信息对象          Configuration   configuration = **new** Configuration();                           FileSystem fs =   FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                  //     fs.copyToLocalFile(new   Path("hdfs://hadoop102:9000/user/atguigu/hello.txt"),   new Path("d:/hello.txt"));          // boolean delSrc 指是否将原文件删除          // Path src 指要下载的文件路径          // Path dst 指将文件下载到的路径          // boolean   useRawLocalFileSystem 是否开启文件效验       // 2 下载文件          fs.copyToLocalFile(**false**,   **new** Path("hdfs://hadoop102:9000/user/atguigu/hello.txt"), **new**   Path("e:/hellocopy.txt"), **true**);          fs.close();          }   

### 3.2.4 HDFS目录创建

​          @Test          **public**   **void** mkdirAtHDFS() **throws** Exception{                 //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                                  FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                       //2   创建目录                 fs.mkdirs(**new**   Path("hdfs://hadoop102:9000/user/atguigu/output"));          }   

### 3.2.5 HDFS文件夹删除

​          @Test          **public**   **void** deleteAtHDFS() **throws** Exception{                 //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                                  FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                       //2   删除文件夹 ，如果是非空文件夹，参数2必须给值true                 fs.delete(**new**   Path("hdfs://hadoop102:9000/user/atguigu/output"), **true**);          }   

### 3.2.6 HDFS文件名更改

​          @Test          **public**   **void** renameAtHDFS() **throws** Exception{                 //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                                  FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                  //2   重命名文件或文件夹                 fs.rename(**new**   Path("hdfs://hadoop102:9000/user/atguigu/hello.txt"), **new**   Path("hdfs://hadoop102:9000/user/atguigu/hellonihao.txt"));          }   

### 3.2.7 HDFS文件详情查看

   @Test   **public** **void** readListFiles() **throws**   Exception {          // 1 创建配置信息对象          Configuration   configuration = **new** Configuration();                           FileSystem fs =   FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                           // 思考：为什么返回迭代器，而不是List之类的容器          RemoteIterator<LocatedFileStatus>   listFiles = fs.listFiles(**new** Path("/"), **true**);              **while**   (listFiles.hasNext()) {                 LocatedFileStatus   fileStatus = listFiles.next();                                         System.**out**.println(fileStatus.getPath().getName());                 System.**out**.println(fileStatus.getBlockSize());                 System.**out**.println(fileStatus.getPermission());                 System.**out**.println(fileStatus.getLen());                                         BlockLocation[]   blockLocations = fileStatus.getBlockLocations();                                         **for**   (BlockLocation bl : blockLocations) {                                                       System.**out**.println("block-offset:"   + bl.getOffset());                                                       String[]   hosts = bl.getHosts();                                                       **for**   (String host : hosts) {                               System.**out**.println(host);                        }                 }                                         System.**out**.println("--------------李冰冰的分割线--------------");          }          }   

### 3.2.8 HDFS文件夹查看

   @Test   public void findAtHDFS() throws Exception,   IllegalArgumentException, IOException{                           // 1 创建配置信息对象          Configuration   configuration = new Configuration();                           FileSystem fs =   FileSystem.get(new URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                           // 2 获取查询路径下的文件状态信息          FileStatus[]   listStatus = fs.listStatus(new Path("/"));              // 3 遍历所有文件状态          **for** (FileStatus   status : listStatus) {                 **if**   (status.isFile()) {                        System.**out**.println("f--"   + status.getPath().getName());                 } **else** {                        System.**out**.println("d--"   + status.getPath().getName());                 }          }   }   

## 3.3 通过IO流操作HDFS

### 3.3.1 HDFS文件上传

​          @Test          **public**   **void** putFileToHDFS() **throws** Exception{                 //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                                  FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                  //   2 创建输入流                 FileInputStream   inStream = **new** FileInputStream(**new**   File("e:/hello.txt"));                                  //   3 获取输出路径                 String   putFileName = "hdfs://hadoop102:9000/user/atguigu/hello1.txt";                 Path   writePath = **new** Path(putFileName);                     //   4 创建输出流                 FSDataOutputStream   outStream = fs.create(writePath);                     //   5 流对接                 **try**{                        IOUtils.*copyBytes*(inStream,   outStream, 4096, **false**);                 }**catch**(Exception   e){                        e.printStackTrace();                 }**finally**{                        IOUtils.*closeStream*(inStream);                        IOUtils.*closeStream*(outStream);                 }          }   

### 3.3.2 HDFS文件下载

​          @Test          **public**   **void** getFileToHDFS() **throws** Exception{                 //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                                  FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),configuration,   "atguigu");                                  //   2 获取读取文件路径                 String   filename = "hdfs://hadoop102:9000/user/atguigu/hello1.txt";                                  //   3 创建读取path                 Path   readPath = **new** Path(filename);                                  //   4 创建输入流                 FSDataInputStream   inStream = fs.open(readPath);                                  //   5 流对接输出到控制台                 **try**{                        IOUtils.*copyBytes*(inStream,   System.**out**, 4096, **false**);                 }**catch**(Exception   e){                        e.printStackTrace();                 }**finally**{                        IOUtils.*closeStream*(inStream);                 }          }   

### 3.3.3 定位文件读取

1）下载第一块

   @Test   // 定位下载第一块内容   **public** **void** readFileSeek1() **throws**   Exception {              // 1 创建配置信息对象          Configuration   configuration = **new** Configuration();              FileSystem fs =   FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),   configuration, "atguigu");              // 2 获取输入流路径          Path path = **new**   Path("hdfs://hadoop102:9000/user/atguigu/tmp/hadoop-2.7.2.tar.gz");              // 3 打开输入流          FSDataInputStream fis   = fs.open(path);              // 4 创建输出流          FileOutputStream fos =   **new** FileOutputStream("e:/hadoop-2.7.2.tar.gz.part1");              // 5 流对接          **byte**[] buf = **new**   **byte**[1024];          **for** (**int**   i = 0; i < 128 * 1024; i++) {                 fis.read(buf);                 fos.write(buf);          }              // 6 关闭流          IOUtils.*closeStream*(fis);          IOUtils.*closeStream*(fos);          }   

2）下载第二块

​          @Test          //   定位下载第二块内容          **public**   **void** readFileSeek2() **throws** Exception{                                  //   1 创建配置信息对象                 Configuration   configuration = **new** Configuration();                     FileSystem   fs = FileSystem.*get*(**new** URI("hdfs://hadoop102:9000"),   configuration, "atguigu");                                  //   2 获取输入流路径                 Path   path = **new** Path("hdfs://hadoop102:9000/user/atguigu/tmp/hadoop-2.7.2.tar.gz");                                  //   3 打开输入流                 FSDataInputStream   fis = fs.open(path);                                  //   4 创建输出流                 FileOutputStream   fos = **new** FileOutputStream("e:/hadoop-2.7.2.tar.gz.part2");                                  //   5 定位偏移量（第二块的首位）                 fis.seek(1024   * 1024 * 128);                                  //   6 流对接                 IOUtils.*copyBytes*(fis,   fos, 1024);                                  //   7 关闭流                 IOUtils.*closeStream*(fis);                 IOUtils.*closeStream*(fos);          }   

3）合并文件

在window命令窗口中执行

type hadoop-2.7.2.tar.gz.part2 >> hadoop-2.7.2.tar.gz.part1

# 四 HDFS的数据流

## 4.1 HDFS写数据流程

### 4.1.1 剖析文件写入

![img](D:\software\Typora\imgs\clip_image008-1544403202080.png)

1）客户端向namenode请求上传文件，namenode检查目标文件是否已存在，父目录是否存在。

2）namenode返回是否可以上传。

3）客户端请求第一个 block上传到哪几个datanode服务器上。

4）namenode返回3个datanode节点，分别为dn1、dn2、dn3。

5）客户端请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成

6）dn1、dn2、dn3逐级应答客户端

7）客户端开始往dn1上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，dn1收到一个packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答

8）当一个block传输完成之后，客户端再次请求namenode上传第二个block的服务器。（重复执行3-7步）

### 4.1.2 网络拓扑概念

​       在本地网络中，两个节点被称为“彼此近邻”是什么意思？在海量数据处理中，其主要限制因素是节点之间数据的传输速率——带宽很稀缺。这里的想法是将两个节点间的带宽作为距离的衡量标准。

​       节点距离：两个节点到达最近的共同祖先的距离总和。

例如，假设有数据中心d1机架r1中的节点n1。该节点可以表示为/d1/r1/n1。利用这种标记，这里给出四种距离描述。

Distance(/d1/r1/n1, /d1/r1/n1)=0（同一节点上的进程）

Distance(/d1/r1/n1, /d1/r1/n2)=2（同一机架上的不同节点）

Distance(/d1/r1/n1, /d1/r3/n2)=4（同一数据中心不同机架上的节点）

Distance(/d1/r1/n1, /d2/r4/n2)=6（不同数据中心的节点）

![img](D:\software\Typora\imgs\clip_image010.png)

大家算一算每两个节点之间的距离。

![img](D:\software\Typora\imgs\clip_image012-1544403202080.jpg)

### 4.1.3 机架感知（副本节点选择）

1）官方ip地址：

<http://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-common/RackAwareness.html>

http://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Data_Replication

2）低版本Hadoop副本节点选择

第一个副本在client所处的节点上。如果客户端在集群外，随机选一个。

第二个副本和第一个副本位于不相同机架的随机节点上。

第三个副本和第二个副本位于相同机架，节点随机。

![img](D:\software\Typora\imgs\clip_image014-1544403202080.png)

3）Hadoop2.7.2副本节点选择

​       第一个副本在client所处的节点上。如果客户端在集群外，随机选一个。

​       第二个副本和第一个副本位于相同机架，随机节点。

​       第三个副本位于不同机架，随机节点。

![img](D:\software\Typora\imgs\clip_image016-1544403202080.png)

## 4.2 HDFS读数据流程

![img](D:\software\Typora\imgs\clip_image018-1544403202080.png)

1）客户端向namenode请求下载文件，namenode通过查询元数据，找到文件块所在的datanode地址。

2）挑选一台datanode（就近原则，然后随机）服务器，请求读取数据。

3）datanode开始传输数据给客户端（从磁盘里面读取数据放入流，以packet为单位来做校验）。

4）客户端以packet为单位接收，先在本地缓存，然后写入目标文件。

## 4.3 一致性模型

1）debug调试如下代码

   @Test          public   void writeFile() throws Exception{                 //   1 创建配置信息对象                 Configuration   configuration = new Configuration();                 fs   = FileSystem.get(configuration);                                  //   2 创建文件输出流                 Path   path = new Path("hdfs://hadoop102:9000/user/atguigu/hello.txt");                 FSDataOutputStream   fos = fs.create(path);                                  //   3 写数据                 fos.write("hello".getBytes());             // 4 一致性刷新                 fos.hflush();                                  fos.close();          }   

2）总结

写入数据时，如果希望数据被其他client立即可见，调用如下方法

FsDataOutputStream. hflush ();            //清理客户端缓冲区数据，被其他client立即可见

# 五 NameNode工作机制

## 5.1 NameNode&Secondary NameNode工作机制

![img](D:\software\Typora\imgs\clip_image020-1544403202080.png)1）第一阶段：namenode启动

（1）第一次启动namenode格式化后，创建fsimage和edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。

（2）客户端对元数据进行增删改的请求

（3）namenode记录操作日志，更新滚动日志。

（4）namenode在内存中对数据进行增删改查

2）第二阶段：Secondary NameNode工作

​       （1）Secondary NameNode询问namenode是否需要checkpoint。直接带回namenode是否检查结果。

​       （2）Secondary NameNode请求执行checkpoint。

​       （3）namenode滚动正在写的edits日志

​       （4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode

​       （5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。

​       （6）生成新的镜像文件fsimage.chkpoint

​       （7）拷贝fsimage.chkpoint到namenode

​       （8）namenode将fsimage.chkpoint重新命名成fsimage

3）web端访问SecondaryNameNode

​       （1）启动集群

​       （2）浏览器中输入：http://hadoop102:50090/status.html

​       （3）查看SecondaryNameNode信息

![img](D:\software\Typora\imgs\clip_image022-1544403202080.jpg)

4）chkpoint检查时间参数设置

（1）通常情况下，SecondaryNameNode每隔一小时执行一次。

​       [hdfs-default.xml]

   <property>       <name>dfs.namenode.checkpoint.period</name>     <value>3600</value>   </property>   

（2）一分钟检查一次操作次数，当操作次数达到1百万时，SecondaryNameNode执行一次。

   <property>       <name>dfs.namenode.checkpoint.txns</name>     <value>1000000</value>   <description>操作动作次数</description>   </property>       <property>       <name>dfs.namenode.checkpoint.check.period</name>     <value>60</value>   <description> 1分钟检查一次操作次数</description>   </property>   

## 5.2 镜像文件和编辑日志文件

1）概念

​       namenode被格式化之后，将在/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current目录中产生如下文件

   edits_0000000000000000000   fsimage_0000000000000000000.md5   seen_txid   VERSION   

（1）Fsimage文件：HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目录和文件idnode的序列化信息。 

（2）Edits文件：存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先会被记录到edits文件中。 

（3）seen_txid文件保存的是一个数字，就是最后一个edits_的数字

（4）每次Namenode启动的时候都会将fsimage文件读入内存，并从00001开始到seen_txid中记录的数字依次执行每个edits里面的更新操作，保证内存中的元数据信息是最新的、同步的，可以看成Namenode启动的时候就将fsimage和edits文件进行了合并。

2）oiv查看fsimage文件

（1）查看oiv和oev命令

[atguigu@hadoop102 current]$ hdfs

oiv                  apply the offline fsimage viewer to an fsimage

oev                  apply the offline edits viewer to an edits file

（2）基本语法

hdfs oiv -p 文件类型 -i镜像文件 -o 转换后文件输出路径

（3）案例实操

[atguigu@hadoop102 current]$ pwd

/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current

 

[atguigu@hadoop102 current]$ hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-2.7.2/fsimage.xml

 

[atguigu@hadoop102 current]$ cat /opt/module/hadoop-2.7.2/fsimage.xml

将显示的xml文件内容拷贝到eclipse中创建的xml文件中，并格式化。

3）oev查看edits文件

（1）基本语法

hdfs oev -p 文件类型 -i编辑日志 -o 转换后文件输出路径

（2）案例实操

[atguigu@hadoop102 current]$ hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-2.7.2/edits.xml

[atguigu@hadoop102 current]$ cat /opt/module/hadoop-2.7.2/edits.xml

将显示的xml文件内容拷贝到eclipse中创建的xml文件中，并格式化。

## 5.3 滚动编辑日志

正常情况HDFS文件系统有更新操作时，就会滚动编辑日志。也可以用命令强制滚动编辑日志。

1）滚动编辑日志（前提必须启动集群）

[atguigu@hadoop102 current]$ hdfs dfsadmin -rollEdits

2）镜像文件什么时候产生

Namenode启动时加载镜像文件和编辑日志

![img](D:\software\Typora\imgs\clip_image024-1544403202080.jpg)

## 5.4 namenode版本号

1）查看namenode版本号

在/opt/module/hadoop-2.7.2/data/tmp/dfs/name/current这个目录下查看VERSION

namespaceID=1933630176

clusterID=CID-1f2bf8d1-5ad2-4202-af1c-6713ab381175

cTime=0

storageType=NAME_NODE

blockpoolID=BP-97847618-192.168.10.102-1493726072779

layoutVersion=-63

2）namenode版本号具体解释

（1）namespaceID在HDFS上，会有多个Namenode，所以不同Namenode的namespaceID是不同的，分别管理一组blockpoolID。

（2）clusterID集群id，全局唯一

（3）cTime属性标记了namenode存储系统的创建时间，对于刚刚格式化的存储系统，这个属性为0；但是在文件系统升级之后，该值会更新到新的时间戳。

（4）storageType属性说明该存储目录包含的是namenode的数据结构。

（5）blockpoolID：一个block pool id标识一个block pool，并且是跨集群的全局唯一。当一个新的Namespace被创建的时候(format过程的一部分)会创建并持久化一个唯一ID。在创建过程构建全局唯一的BlockPoolID比人为的配置更可靠一些。NN将BlockPoolID持久化到磁盘中，在后续的启动过程中，会再次load并使用。

（6）layoutVersion是一个负整数。通常只有HDFS增加新特性时才会更新这个版本号。

## 5.5 SecondaryNameNode目录结构

Secondary NameNode用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

在/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/current这个目录中查看SecondaryNameNode目录结构。

   edits_0000000000000000001-0000000000000000002   fsimage_0000000000000000002   fsimage_0000000000000000002.md5   VERSION   

SecondaryNameNode的namesecondary/current目录和主namenode的current目录的布局相同。

好处：在主namenode发生故障时（假设没有及时备份数据），可以从SecondaryNameNode恢复数据。

方法一：将SecondaryNameNode中数据拷贝到namenode存储数据的目录；

方法二：使用-importCheckpoint选项启动namenode守护进程，从而将SecondaryNameNode用作新的主namenode。

1）案例实操（一）：

模拟namenode故障，并采用方法一，恢复namenode数据

（1）kill -9 namenode进程

（2）删除namenode存储的数据（/opt/module/hadoop-2.7.2/data/tmp/dfs/name）

rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*

（3）拷贝SecondaryNameNode中数据到原namenode存储数据目录

​         cp -R /opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/* /opt/module/hadoop-2.7.2/data/tmp/dfs/name/

（4）重新启动namenode

sbin/hadoop-daemon.sh start namenode

2）案例实操（二）：

模拟namenode故障，并采用方法二，恢复namenode数据

（0）修改hdfs-site.xml中的

   <property>       <name>dfs.namenode.checkpoint.period</name>       <value>120</value>   </property>       <property>       <name>dfs.namenode.name.dir</name>       <value>/opt/module/hadoop-2.7.2/data/tmp/dfs/name</value>   </property>   

（1）kill -9 namenode进程

（2）删除namenode存储的数据（/opt/module/hadoop-2.7.2/data/tmp/dfs/name）

rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*

（3）如果SecondaryNameNode不和Namenode在一个主机节点上，需要将SecondaryNameNode存储数据的目录拷贝到Namenode存储数据的平级目录。

   [atguigu@hadoop102 dfs]$ pwd   /opt/module/hadoop-2.7.2/data/tmp/dfs   [atguigu@hadoop102 dfs]$ ls   data    name  namesecondary   

（4）导入检查点数据（等待一会ctrl+c结束掉）

bin/hdfs namenode -importCheckpoint

（5）启动namenode

sbin/hadoop-daemon.sh start namenode

（6）如果提示文件锁了，可以删除in_use.lock 

​              rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/in_use.lock

## 5.6 集群安全模式操作

1）概述

Namenode启动时，首先将映像文件（fsimage）载入内存，并执行编辑日志（edits）中的各项操作。一旦在内存中成功建立文件系统元数据的映像，则创建一个新的fsimage文件和一个空的编辑日志。此时，namenode开始监听datanode请求。但是此刻，namenode运行在安全模式，即namenode的文件系统对于客户端来说是只读的。

系统中的数据块的位置并不是由namenode维护的，而是以块列表的形式存储在datanode中。在系统的正常操作期间，namenode会在内存中保留所有块位置的映射信息。在安全模式下，各个datanode会向namenode发送最新的块列表信息，namenode了解到足够多的块位置信息之后，即可高效运行文件系统。

如果满足“最小副本条件”，namenode会在30秒钟之后就退出安全模式。所谓的最小副本条件指的是在整个文件系统中99.9%的块满足最小副本级别（默认值：dfs.replication.min=1）。在启动一个刚刚格式化的HDFS集群时，因为系统中还没有任何块，所以namenode不会进入安全模式。

2）基本语法

集群处于安全模式，不能执行重要操作（写操作）。集群启动完成后，自动退出安全模式。

（1）bin/hdfs dfsadmin -safemode get        （功能描述：查看安全模式状态）

（2）bin/hdfs dfsadmin -safemode enter    （功能描述：进入安全模式状态）

（3）bin/hdfs dfsadmin -safemode leave     （功能描述：离开安全模式状态）

（4）bin/hdfs dfsadmin -safemode wait       （功能描述：等待安全模式状态）

3）案例

​       模拟等待安全模式

​       1）先进入安全模式

bin/hdfs dfsadmin -safemode enter

​       2）执行下面的脚本

编辑一个脚本

   #!/bin/bash   bin/hdfs dfsadmin -safemode wait   bin/hdfs dfs -put ~/hello.txt   /root/hello.txt   

​       3）再打开一个窗口，执行

bin/hdfs dfsadmin -safemode leave

## 5.7 Namenode多目录配置

1）namenode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性。

2）具体配置如下：

​       hdfs-site.xml

   <property>         <name>dfs.namenode.name.dir</name>   <value>file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2</value>   </property>   

# 六 DataNode工作机制

## 6.1 DataNode工作机制

![img](D:\software\Typora\imgs\clip_image026-1544403202080.png)

1）一个数据块在datanode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。

2）DataNode启动后向namenode注册，通过后，周期性（1小时）的向namenode上报所有的块信息。

3）心跳是每3秒一次，心跳返回结果带有namenode给该datanode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个datanode的心跳，则认为该节点不可用。

4）集群运行中可以安全加入和退出一些机器

## 6.2 数据完整性

1）当DataNode读取block的时候，它会计算checksum

2）如果计算后的checksum，与block创建时值不一样，说明block已经损坏。

3）client读取其他DataNode上的block.

4）datanode在其文件创建后周期验证checksum

## 6.3 掉线时限参数设置

datanode进程死亡或者网络故障造成datanode无法与namenode通信，namenode不会立即把该节点判定为死亡，要经过一段时间，这段时间暂称作超时时长。HDFS默认的超时时长为10分钟+30秒。如果定义超时时间为timeout，则超时时长的计算公式为：

​       timeout  = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval。

​       而默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。

​       需要注意的是hdfs-site.xml 配置文件中的heartbeat.recheck.interval的单位为毫秒，dfs.heartbeat.interval的单位为秒。

   <property>       <name>dfs.namenode.heartbeat.recheck-interval</name>       <value>300000</value>   </property>   <property>       <name> dfs.heartbeat.interval </name>       <value>3</value>   </property>   

## 6.4 DataNode的目录结构

和namenode不同的是，datanode的存储目录是初始阶段自动创建的，不需要额外格式化。

1）在/opt/module/hadoop-2.7.2/data/tmp/dfs/data/current这个目录下查看版本号

[atguigu@hadoop102 current]$ cat VERSION 

storageID=DS-1b998a1d-71a3-43d5-82dc-c0ff3294921b

clusterID=CID-1f2bf8d1-5ad2-4202-af1c-6713ab381175

cTime=0

datanodeUuid=970b2daf-63b8-4e17-a514-d81741392165

storageType=DATA_NODE

layoutVersion=-56

2）具体解释

​       （1）storageID：存储id号

​       （2）clusterID集群id，全局唯一

​       （3）cTime属性标记了datanode存储系统的创建时间，对于刚刚格式化的存储系统，这个属性为0；但是在文件系统升级之后，该值会更新到新的时间戳。

​       （4）datanodeUuid：datanode的唯一识别码

​       （5）storageType：存储类型

​       （6）layoutVersion是一个负整数。通常只有HDFS增加新特性时才会更新这个版本号。

3）在/opt/module/hadoop-2.7.2/data/tmp/dfs/data/current/BP-97847618-192.168.10.102-1493726072779/current这个目录下查看该数据块的版本号

[atguigu@hadoop102 current]$ cat VERSION 

\#Mon May 08 16:30:19 CST 2017

namespaceID=1933630176

cTime=0

blockpoolID=BP-97847618-192.168.10.102-1493726072779

layoutVersion=-56

4）具体解释

（1）namespaceID：是datanode首次访问namenode的时候从namenode处获取的storageID对每个datanode来说是唯一的（但对于单个datanode中所有存储目录来说则是相同的），namenode可用这个属性来区分不同datanode。

（2）cTime属性标记了datanode存储系统的创建时间，对于刚刚格式化的存储系统，这个属性为0；但是在文件系统升级之后，该值会更新到新的时间戳。

（3）blockpoolID：一个block pool id标识一个block pool，并且是跨集群的全局唯一。当一个新的Namespace被创建的时候(format过程的一部分)会创建并持久化一个唯一ID。在创建过程构建全局唯一的BlockPoolID比人为的配置更可靠一些。NN将BlockPoolID持久化到磁盘中，在后续的启动过程中，会再次load并使用。

（4）layoutVersion是一个负整数。通常只有HDFS增加新特性时才会更新这个版本号。

## 6.5 服役新数据节点

0）需求：

随着公司业务的增长，数据量越来越大，原有的数据节点的容量已经不能满足存储数据的需求，需要在原有集群基础上动态添加新的数据节点。

1）环境准备

​       （1）克隆一台虚拟机

​       （2）修改ip地址和主机名称

​       （3）修改xcall和xsync文件，增加新增节点的同步

​       （4）删除原来HDFS文件系统留存的文件

​              /opt/module/hadoop-2.7.2/data

2）服役新节点具体步骤

​       （1）在namenode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts文件

[atguigu@hadoop105 hadoop]$ pwd

/opt/module/hadoop-2.7.2/etc/hadoop

[atguigu@hadoop105 hadoop]$ touch dfs.hosts

[atguigu@hadoop105 hadoop]$ vi dfs.hosts

添加如下主机名称（包含新服役的节点）

hadoop102

hadoop103

hadoop104

hadoop105

​       （2）在namenode的hdfs-site.xml配置文件中增加dfs.hosts属性

   <property>   <name>dfs.hosts</name>           <value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts</value>   </property>   

​       （3）刷新namenode 

[atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes

Refresh nodes successful

​       （4）更新resourcemanager节点

[atguigu@hadoop102 hadoop-2.7.2]$ yarn rmadmin -refreshNodes

17/06/24 14:17:11 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033

​       （5）在namenode的slaves文件中增加新主机名称

​              增加105  不需要分发

hadoop102

hadoop103

hadoop104

hadoop105

​       （6）单独命令启动新的数据节点和节点管理器

[atguigu@hadoop105 hadoop-2.7.2]$ sbin/hadoop-daemon.sh start datanode

starting datanode, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-datanode-hadoop105.out

[atguigu@hadoop105 hadoop-2.7.2]$ sbin/yarn-daemon.sh start nodemanager

starting nodemanager, logging to /opt/module/hadoop-2.7.2/logs/yarn-atguigu-nodemanager-hadoop105.out

​       （7）在web浏览器上检查是否ok

3）如果数据不均衡，可以用命令实现集群的再平衡

​       [atguigu@hadoop102 sbin]$ ./start-balancer.sh

starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out

Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved

## 6.6 退役旧数据节点

1）在namenode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts.exclude文件

​       [atguigu@hadoop102 hadoop]$ pwd

/opt/module/hadoop-2.7.2/etc/hadoop

[atguigu@hadoop102 hadoop]$ touch dfs.hosts.exclude

[atguigu@hadoop102 hadoop]$ vi dfs.hosts.exclude

添加如下主机名称（要退役的节点）

hadoop105

2）在namenode的hdfs-site.xml配置文件中增加dfs.hosts.exclude属性

   <property>   <name>dfs.hosts.exclude</name>         <value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts.exclude</value>   </property>   

3）刷新namenode、刷新resourcemanager

[atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes

Refresh nodes successful

[atguigu@hadoop102 hadoop-2.7.2]$ yarn rmadmin -refreshNodes

17/06/24 14:55:56 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033

4）检查web浏览器，退役节点的状态为decommission in progress（退役中），说明数据节点正在复制块到其他节点。

![img](D:\software\Typora\imgs\clip_image028-1544403202080.jpg)

5）等待退役节点状态为decommissioned（所有块已经复制完成），停止该节点及节点资源管理器。注意：如果副本数是3，服役的节点小于等于3，是不能退役成功的，需要修改副本数后才能退役。·

![img](D:\software\Typora\imgs\clip_image030-1544403202080.jpg)

[atguigu@hadoop105 hadoop-2.7.2]$ sbin/hadoop-daemon.sh stop datanode

stopping datanode

[atguigu@hadoop105 hadoop-2.7.2]$ sbin/yarn-daemon.sh stop nodemanager

stopping nodemanager

6）从include文件中删除退役节点，再运行刷新节点的命令

​       （1）从namenode的dfs.hosts文件中删除退役节点hadoop105

hadoop102

hadoop103

hadoop104

​       （2）刷新namenode，刷新resourcemanager

[atguigu@hadoop102 hadoop-2.7.2]$ hdfs dfsadmin -refreshNodes

Refresh nodes successful

[atguigu@hadoop102 hadoop-2.7.2]$ yarn rmadmin -refreshNodes

17/06/24 14:55:56 INFO client.RMProxy: Connecting to ResourceManager at hadoop103/192.168.1.103:8033

7）从namenode的slave文件中删除退役节点hadoop105

hadoop102

hadoop103

hadoop104

8）如果数据不均衡，可以用命令实现集群的再平衡

[atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-balancer.sh 

starting balancer, logging to /opt/module/hadoop-2.7.2/logs/hadoop-atguigu-balancer-hadoop102.out

Time Stamp               Iteration#  Bytes Already Moved  Bytes Left To Move  Bytes Being Moved

## 6.7 Datanode多目录配置

1）datanode也可以配置成多个目录，每个目录存储的数据不一样。即：数据不是副本。

2）具体配置如下：

​       hdfs-site.xml

​          <property>           <name>dfs.datanode.data.dir</name>             <value>file:///${hadoop.tmp.dir}/dfs/data1,file:///${hadoop.tmp.dir}/dfs/data2</value>       </property>   

 

# 七 HDFS其他功能

## 7.1 集群间数据拷贝

1）scp实现两个远程主机之间的文件复制

​       scp -r hello.txt [root@hadoop103:/user/atguigu/hello.txt](mailto:root@hadoop103:/user/atguigu/hello.txt)             // 推 push

​       scp -r [root@hadoop103:/user/atguigu/hello.txt  hello.txt](mailto:root@hadoop103:/user/atguigu/hello.txt%20%20hello.txt)           // 拉 pull

​       scp -r [root@hadoop103:/user/atguigu/hello.txt](mailto:root@hadoop103:/user/atguigu/hello.txt) root@hadoop104:/user/atguigu   //是通过本地主机中转实现两个远程主机的文件复制；如果在两个远程主机之间ssh没有配置的情况下可以使用该方式。

2）采用discp命令实现两个hadoop集群之间的递归数据复制

bin/hadoop distcp hdfs://haoop102:9000/user/atguigu/hello.txt hdfs://hadoop103:9000/user/atguigu/hello.txt

## 7.2 Hadoop存档

1）理论概述

每个文件均按块存储，每个块的元数据存储在namenode的内存中，因此hadoop存储小文件会非常低效。因为大量的小文件会耗尽namenode中的大部分内存。但注意，存储小文件所需要的磁盘容量和存储这些文件原始内容所需要的磁盘空间相比也不会增多。例如，一个1MB的文件以大小为128MB的块存储，使用的是1MB的磁盘空间，而不是128MB。

Hadoop存档文件或HAR文件，是一个更高效的文件存档工具，它将文件存入HDFS块，在减少namenode内存使用的同时，允许对文件进行透明的访问。具体说来，Hadoop存档文件可以用作MapReduce的输入。

2）案例实操

（1）需要启动yarn进程

​       start-yarn.sh

（2）归档文件

​       归档成一个叫做xxx.har的文件夹，该文件夹下有相应的数据文件。Xx.har目录是一个整体，该目录看成是一个归档文件即可。

bin/hadoop archive -archiveName myhar.har -p /user/atguigu   /user/my

（3）查看归档

​       hadoop fs -lsr /user/my/myhar.har

hadoop fs -lsr har:///myhar.har

（4）解归档文件

hadoop fs -cp har:/// user/my/myhar.har /* /user/atguigu

## 7.3 快照管理

快照相当于对目录做一个备份。并不会立即复制所有文件，而是指向同一个文件。当写入发生时，才会产生新文件。

1）基本语法

​       （1）hdfs dfsadmin -allowSnapshot 路径   （功能描述：开启指定目录的快照功能）

​       （2）hdfs dfsadmin -disallowSnapshot 路径 （功能描述：禁用指定目录的快照功能，默认是禁用）

​       （3）hdfs dfs -createSnapshot 路径        （功能描述：对目录创建快照）

​       （4）hdfs dfs -createSnapshot 路径 名称   （功能描述：指定名称创建快照）

​       （5）hdfs dfs -renameSnapshot 路径 旧名称 新名称 （功能描述：重命名快照）

​       （6）hdfs lsSnapshottableDir         （功能描述：列出当前用户所有可快照目录）

​       （7）hdfs snapshotDiff 路径1 路径2 （功能描述：比较两个快照目录的不同之处）

​       （8）hdfs dfs -deleteSnapshot <path> <snapshotName>  （功能描述：删除快照）

2）案例实操

​       （1）开启/禁用指定目录的快照功能

hdfs dfsadmin -allowSnapshot /user/atguigu/data         

hdfs dfsadmin -disallowSnapshot /user/atguigu/data     

​       （2）对目录创建快照

hdfs dfs -createSnapshot /user/atguigu/data          // 对目录创建快照

通过web访问hdfs://hadoop102:9000/user/atguigu/data/.snapshot/s…..// 快照和源文件使用相同数据块

hdfs dfs -lsr /user/atguigu/data/.snapshot/

​       （3）指定名称创建快照

hdfs dfs -createSnapshot /user/atguigu/data miao170508            

​       （4）重命名快照

hdfs dfs -renameSnapshot /user/atguigu/data/ miao170508 atguigu170508         

​       （5）列出当前用户所有可快照目录

hdfs lsSnapshottableDir  

​       （6）比较两个快照目录的不同之处

hdfs snapshotDiff /user/atguigu/data/  .  .snapshot/atguigu170508   

​       （7）恢复快照

hdfs dfs -cp /user/atguigu/input/.snapshot/s20170708-134303.027 /user

## 7.4 回收站

1）默认回收站

默认值fs.trash.interval=0，0表示禁用回收站，可以设置删除文件的存活时间。

默认值fs.trash.checkpoint.interval=0，检查回收站的间隔时间。

要求fs.trash.checkpoint.interval<=fs.trash.interval。

![img](D:\software\Typora\imgs\clip_image032-1544403202080.jpg)

2）启用回收站

修改core-site.xml，配置垃圾回收时间为1分钟。

   <property>         <name>fs.trash.interval</name>       <value>1</value>   </property>   

3）查看回收站

回收站在集群中的；路径：/user/atguigu/.Trash/….

4）修改访问垃圾回收站用户名称

​       进入垃圾回收站用户名称，默认是dr.who，修改为atguigu用户

​       [core-site.xml]

   <property>       <name>hadoop.http.staticuser.user</name>     <value>atguigu</value>   </property>   

5）通过程序删除的文件不会经过回收站，需要调用moveToTrash()才进入回收站

Trash trash = New Trash(conf);

trash.moveToTrash(path);

6）恢复回收站数据

hadoop fs -mv /user/atguigu/.Trash/Current/user/atguigu/input    /user/atguigu/input

7）清空回收站

hdfs dfs -expunge

# 八 HDFS HA高可用

## 8.1 HA概述

1）所谓HA（high available），即高可用（7*24小时不中断服务）。

2）实现高可用最关键的策略是消除单点故障。HA严格来说应该分成各个组件的HA机制：HDFS的HA和YARN的HA。

3）Hadoop2.0之前，在HDFS集群中NameNode存在单点故障（SPOF）。

4）NameNode主要在以下两个方面影响HDFS集群

​       NameNode机器发生意外，如宕机，集群将无法使用，直到管理员重启

​       NameNode机器需要升级，包括软件、硬件升级，此时集群也将无法使用

HDFS HA功能通过配置Active/Standby两个nameNodes实现在集群中对NameNode的热备来解决上述问题。如果出现故障，如机器崩溃或机器需要升级维护，这时可通过此种方式将NameNode很快的切换到另外一台机器。

## 8.2 HDFS-HA工作机制

1）通过双namenode消除单点故障

### 8.2.1 HDFS-HA工作要点

1）元数据管理方式需要改变：

内存中各自保存一份元数据；

Edits日志只有Active状态的namenode节点可以做写操作；

两个namenode都可以读取edits；

共享的edits放在一个共享存储中管理（qjournal和NFS两个主流实现）；

2）需要一个状态管理功能模块

实现了一个zkfailover，常驻在每一个namenode所在的节点，每一个zkfailover负责监控自己所在namenode节点，利用zk进行状态标识，当需要进行状态切换时，由zkfailover来负责切换，切换时需要防止brain split现象的发生。

3）必须保证两个NameNode之间能够ssh无密码登录。

4）隔离（Fence），即同一时刻仅仅有一个NameNode对外提供服务

### 8.2.2 HDFS-HA自动故障转移工作机制

前面学习了使用命令hdfs haadmin -failover手动进行故障转移，在该模式下，即使现役NameNode已经失效，系统也不会自动从现役NameNode转移到待机NameNode，下面学习如何配置部署HA自动进行故障转移。自动故障转移为HDFS部署增加了两个新组件：ZooKeeper和ZKFailoverController（ZKFC）进程。ZooKeeper是维护少量协调数据，通知客户端这些数据的改变和监视客户端故障的高可用服务。HA的自动故障转移依赖于ZooKeeper的以下功能：

**1****）故障检测：**集群中的每个NameNode在ZooKeeper中维护了一个持久会话，如果机器崩溃，ZooKeeper中的会话将终止，ZooKeeper通知另一个NameNode需要触发故障转移。

**2****）现役NameNode****选择：**ZooKeeper提供了一个简单的机制用于唯一的选择一个节点为active状态。如果目前现役NameNode崩溃，另一个节点可能从ZooKeeper获得特殊的排外锁以表明它应该成为现役NameNode。

ZKFC是自动故障转移中的另一个新组件，是ZooKeeper的客户端，也监视和管理NameNode的状态。每个运行NameNode的主机也运行了一个ZKFC进程，ZKFC负责：

**1****）健康监测：**ZKFC使用一个健康检查命令定期地ping与之在相同主机的NameNode，只要该NameNode及时地回复健康状态，ZKFC认为该节点是健康的。如果该节点崩溃，冻结或进入不健康状态，健康监测器标识该节点为非健康的。

**2****）ZooKeeper****会话管理：**当本地NameNode是健康的，ZKFC保持一个在ZooKeeper中打开的会话。如果本地NameNode处于active状态，ZKFC也保持一个特殊的znode锁，该锁使用了ZooKeeper对短暂节点的支持，如果会话终止，锁节点将自动删除。

**3****）基于ZooKeeper****的选择：**如果本地NameNode是健康的，且ZKFC发现没有其它的节点当前持有znode锁，它将为自己获取该锁。如果成功，则它已经赢得了选择，并负责运行故障转移进程以使它的本地NameNode为active。故障转移进程与前面描述的手动故障转移相似，首先如果必要保护之前的现役NameNode，然后本地NameNode转换为active状态。

![img](D:\software\Typora\imgs\clip_image034-1544403202080.png)

## 8.4 HDFS-HA集群配置

### 8.4.1 环境准备

1）修改IP

2）修改主机名及主机名和IP地址的映射

3）关闭防火墙

4）ssh免密登录

5）安装JDK，配置环境变量等

### 8.4.2 规划集群

hadoop102                         hadoop103                        hadoop104             

NameNode                          NameNode

JournalNode                        JournalNode                        JournalNode          

DataNode                            DataNode                            DataNode              

ZK                                     ZK                                     ZK

ResourceManager

NodeManager                      NodeManager                      NodeManager        

### 8.4.3 配置Zookeeper集群

0）集群规划

在hadoop102、hadoop103和hadoop104三个节点上部署Zookeeper。

1）解压安装

（1）解压zookeeper安装包到/opt/module/目录下

 [atguigu@hadoop102 software]$ tar -zxvf zookeeper-3.4.10.tar.gz -C /opt/module/

（2）在/opt/module/zookeeper-3.4.10/这个目录下创建zkData

​       mkdir -p zkData

（3）重命名/opt/module/zookeeper-3.4.10/conf这个目录下的zoo_sample.cfg为zoo.cfg

​       mv zoo_sample.cfg zoo.cfg

2）配置zoo.cfg文件

​       （1）具体配置

​       dataDir=/opt/module/zookeeper-3.4.10/zkData

​       增加如下配置

​       \#######################cluster##########################

server.2=hadoop102:2888:3888

server.3=hadoop103:2888:3888

server.4=hadoop104:2888:3888

（2）配置参数解读

Server.A=B:C:D。

A是一个数字，表示这个是第几号服务器；

B是这个服务器的ip地址；

C是这个服务器与集群中的Leader服务器交换信息的端口；

D是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

集群模式下配置一个文件myid，这个文件在dataDir目录下，这个文件里面有一个数据就是A的值，Zookeeper启动时读取此文件，拿到里面的数据与zoo.cfg里面的配置信息比较从而判断到底是哪个server。

3）集群操作

（1）在/opt/module/zookeeper-3.4.10/zkData目录下创建一个myid的文件

​       touch myid

添加myid文件，注意一定要在linux里面创建，在notepad++里面很可能乱码

（2）编辑myid文件

​       vi myid

​       在文件中添加与server对应的编号：如2

（3）拷贝配置好的zookeeper到其他机器上

​       scp -r zookeeper-3.4.10/ [root@hadoop103.atguigu.com:/opt/app/](mailto:root@hadoop103.atguigu.com:/opt/app/)

​       scp -r zookeeper-3.4.10/ [root@hadoop104.atguigu.com:/opt/app/](mailto:root@hadoop104.atguigu.com:/opt/app/)

​       并分别修改myid文件中内容为3、4

（4）分别启动zookeeper

​       [root@hadoop102 zookeeper-3.4.10]# bin/zkServer.sh start

[root@hadoop103 zookeeper-3.4.10]# bin/zkServer.sh start

[root@hadoop104 zookeeper-3.4.10]# bin/zkServer.sh start

（5）查看状态

[root@hadoop102 zookeeper-3.4.10]# bin/zkServer.sh status

JMX enabled by default

Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg

Mode: follower

[root@hadoop103 zookeeper-3.4.10]# bin/zkServer.sh status

JMX enabled by default

Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg

Mode: leader

[root@hadoop104 zookeeper-3.4.5]# bin/zkServer.sh status

JMX enabled by default

Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg

Mode: follower

### 8.4.4 配置HDFS-HA集群

1）官方地址：<http://hadoop.apache.org/>

2）在opt目录下创建一个ha文件夹

mkdir ha

3）将/opt/app/下的 hadoop-2.7.2拷贝到/opt/ha目录下

cp -r hadoop-2.7.2/ /opt/ha/

4）配置hadoop-env.sh

   export JAVA_HOME=/opt/module/jdk1.7.0_79   

5）配置core-site.xml

   <configuration>   <!-- 把两个NameNode）的地址组装成一个集群mycluster -->                 <property>                        <name>fs.defaultFS</name>                  <value>hdfs://mycluster</value>                 </property>                     <!-- 指定hadoop运行时产生文件的存储目录 -->                 <property>                        <name>hadoop.tmp.dir</name>                        <value>/opt/ha/hadoop-2.7.2/data/tmp</value>                 </property>   </configuration>   

6）配置hdfs-site.xml

   <configuration>          <!--   完全分布式集群名称   -->          <property>                 <name>dfs.nameservices</name>                 <value>mycluster</value>          </property>              <!--   集群中NameNode节点都有哪些 -->          <property>                 <name>dfs.ha.namenodes.mycluster</name>                 <value>nn1,nn2</value>          </property>              <!--   nn1的RPC通信地址 -->          <property>                 <name>dfs.namenode.rpc-address.mycluster.nn1</name>                 <value>hadoop102:9000</value>          </property>              <!--   nn2的RPC通信地址 -->          <property>                 <name>dfs.namenode.rpc-address.mycluster.nn2</name>                 <value>hadoop103:9000</value>          </property>              <!--   nn1的http通信地址 -->          <property>                 <name>dfs.namenode.http-address.mycluster.nn1</name>                 <value>hadoop102:50070</value>          </property>              <!--   nn2的http通信地址 -->          <property>                 <name>dfs.namenode.http-address.mycluster.nn2</name>                 <value>hadoop103:50070</value>          </property>              <!--   指定NameNode元数据在JournalNode上的存放位置 -->          <property>                 <name>dfs.namenode.shared.edits.dir</name>                 <value>qjournal://hadoop102:8485;hadoop103:8485;hadoop104:8485/mycluster</value>          </property>              <!--   配置隔离机制，即同一时刻只能有一台服务器对外响应 -->          <property>                 <name>dfs.ha.fencing.methods</name>                 <value>sshfence</value>          </property>              <!--   使用隔离机制时需要ssh无秘钥登录-->          <property>                 <name>dfs.ha.fencing.ssh.private-key-files</name>                 <value>/home/atguigu/.ssh/id_rsa</value>          </property>              <!--   声明journalnode服务器存储目录-->          <property>                 <name>dfs.journalnode.edits.dir</name>                 <value>/opt/ha/hadoop-2.7.2/data/jn</value>          </property>              <!--   关闭权限检查-->          <property>                 <name>dfs.permissions.enable</name>                 <value>false</value>          </property>              <!--   访问代理类：client，mycluster，active配置失败自动切换实现方式-->          <property>                <name>dfs.client.failover.proxy.provider.mycluster</name>                <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>          </property>   </configuration>   

7）拷贝配置好的hadoop环境到其他节点

### 8.4.5 启动HDFS-HA集群

1）在各个JournalNode节点上，输入以下命令启动journalnode服务：

​       sbin/hadoop-daemon.sh start journalnode

2）在[nn1]上，对其进行格式化，并启动：

​       bin/hdfs namenode -format

​       sbin/hadoop-daemon.sh start namenode

3）在[nn2]上，同步nn1的元数据信息：

​       bin/hdfs namenode -bootstrapStandby

4）启动[nn2]：

​       sbin/hadoop-daemon.sh start namenode

5）查看web页面显示

![img](D:\software\Typora\imgs\clip_image036-1544403202080.jpg)

![img](D:\software\Typora\imgs\clip_image038-1544403202080.jpg)

6）在[nn1]上，启动所有datanode

​       sbin/hadoop-daemons.sh start datanode

7）将[nn1]切换为Active

​       bin/hdfs haadmin -transitionToActive nn1

8）查看是否Active

​       bin/hdfs haadmin -getServiceState nn1

### 8.4.6 配置HDFS-HA自动故障转移

1）具体配置

​       （1）在hdfs-site.xml中增加

   <property>          <name>dfs.ha.automatic-failover.enabled</name>          <value>true</value>   </property>   

​       （2）在core-site.xml文件中增加

   <property>          <name>ha.zookeeper.quorum</name>          <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>   </property>   

2）启动

​       （1）关闭所有HDFS服务：

​              sbin/stop-dfs.sh

​       （2）启动Zookeeper集群：

​              bin/zkServer.sh start

​       （3）初始化HA在Zookeeper中状态：

​              bin/hdfs zkfc -formatZK

​       （4）启动HDFS服务：

​              sbin/start-dfs.sh

​       （5）在各个NameNode节点上启动DFSZK Failover Controller，先在哪台机器启动，哪个机器的NameNode就是Active NameNode

​              sbin/hadoop-daemin.sh start zkfc

3）验证

​       （1）将Active NameNode进程kill

​              kill -9 namenode的进程id

​       （2）将Active NameNode机器断开网络

​              service network stop

## 8.5 YARN-HA配置

### 8.5.1 YARN-HA工作机制

1）官方文档：

<http://hadoop.apache.org/docs/r2.7.3/hadoop-yarn/hadoop-yarn-site/ResourceManagerHA.html>

2）YARN-HA工作机制

![img](D:\software\Typora\imgs\clip_image039.png)

### 8.5.2 配置YARN-HA集群

0）环境准备

（1）修改IP

（2）修改主机名及主机名和IP地址的映射

（3）关闭防火墙

（4）ssh免密登录

（5）安装JDK，配置环境变量等

​       （6）配置Zookeeper集群

1）规划集群

hadoop102                         hadoop103                        hadoop104             

NameNode                          NameNode

JournalNode                        JournalNode                        JournalNode          

DataNode                            DataNode                            DataNode              

ZK                                     ZK                                     ZK

ResourceManager                 ResourceManager

NodeManager                      NodeManager                      NodeManager        

2）具体配置

（1）yarn-site.xml

   <configuration>             <property>             <name>yarn.nodemanager.aux-services</name>             <value>mapreduce_shuffle</value>         </property>             <!--启用resourcemanager   ha-->       <property>             <name>yarn.resourcemanager.ha.enabled</name>             <value>true</value>         </property>             <!--声明两台resourcemanager的地址-->         <property>             <name>yarn.resourcemanager.cluster-id</name>             <value>cluster-yarn1</value>         </property>             <property>             <name>yarn.resourcemanager.ha.rm-ids</name>             <value>rm1,rm2</value>         </property>             <property>             <name>yarn.resourcemanager.hostname.rm1</name>             <value>hadoop102</value>         </property>             <property>             <name>yarn.resourcemanager.hostname.rm2</name>             <value>hadoop103</value>         </property>             <!--指定zookeeper集群的地址-->          <property>             <name>yarn.resourcemanager.zk-address</name>             <value>hadoop102:2181,hadoop103:2181,hadoop104:2181</value>         </property>             <!--启用自动恢复-->            <property>             <name>yarn.resourcemanager.recovery.enabled</name>             <value>true</value>         </property>             <!--指定resourcemanager的状态信息存储在zookeeper集群-->          <property>             <name>yarn.resourcemanager.store.class</name>       <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>   </property>       </configuration>   

​       （2）同步更新其他节点的配置信息

3）启动hdfs 

（1）在各个JournalNode节点上，输入以下命令启动journalnode服务：

​       sbin/hadoop-daemon.sh start journalnode

（2）在[nn1]上，对其进行格式化，并启动：

​       bin/hdfs namenode -format

​       sbin/hadoop-daemon.sh start namenode

（3）在[nn2]上，同步nn1的元数据信息：

​       bin/hdfs namenode -bootstrapStandby

（4）启动[nn2]：

​       sbin/hadoop-daemon.sh start namenode

（5）启动所有datanode

​       sbin/hadoop-daemons.sh start datanode

（6）将[nn1]切换为Active

​       bin/hdfs haadmin -transitionToActive nn1

4）启动yarn 

（1）在hadoop102中执行：

sbin/start-yarn.sh

（2）在hadoop103中执行：

sbin/yarn-daemon.sh start resourcemanager

（3）查看服务状态

bin/yarn rmadmin -getServiceState rm1

 

![img](D:\software\Typora\imgs\clip_image041-1544403202080.jpg)

## 8.6 HDFS Federation架构设计

1）  NameNode架构的局限性

（1）Namespace（命名空间）的限制

由于NameNode在内存中存储所有的元数据（metadata），因此单个namenode所能存储的对象（文件+块）数目受到namenode所在JVM的heap size的限制。50G的heap能够存储20亿（200million）个对象，这20亿个对象支持4000个datanode，12PB的存储（假设文件平均大小为40MB）。随着数据的飞速增长，存储的需求也随之增长。单个datanode从4T增长到36T，集群的尺寸增长到8000个datanode。存储的需求从12PB增长到大于100PB。

（2）隔离问题

由于HDFS仅有一个namenode，无法隔离各个程序，因此HDFS上的一个实验程序就很有可能影响整个HDFS上运行的程序。

​       （3）性能的瓶颈

​       由于是单个namenode的HDFS架构，因此整个HDFS文件系统的吞吐量受限于单个namenode的吞吐量。

2）HDFS Federation架构设计

能不能有多个NameNode

NameNode                                 NameNode                                 NameNode

元数据                                       元数据                                       元数据

Log                                           machine                                     电商数据/话单数据

![img](D:\software\Typora\imgs\clip_image043.png)

3）HDFS Federation应用思考

不同应用可以使用不同NameNode进行数据管理

​        图片业务、爬虫业务、日志审计业务

Hadoop生态系统中，不同的框架使用不同的namenode进行管理namespace。（隔离性）