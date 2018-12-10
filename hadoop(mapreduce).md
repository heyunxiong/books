尚硅谷大数据技术之Hadoop（MapReduce）

(作者：大海哥)

 

官网：[www.atguigu.com](http://www.atguigu.com)

版本：V1.1

# 一 MapReduce概念

Mapreduce是一个分布式运算程序的编程框架，是用户开发“基于hadoop的数据分析应用”的核心框架；

Mapreduce核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个hadoop集群上。

## 1.1 为什么要MapReduce

1）海量数据在单机上处理因为硬件资源限制，无法胜任

2）而一旦将单机版程序扩展到集群来分布式运行，将极大增加程序的复杂度和开发难度

3）引入mapreduce框架后，开发人员可以将绝大部分工作集中在业务逻辑的开发上，而将分布式计算中的复杂性交由框架来处理。

4）mapreduce分布式方案考虑的问题

（1）运算逻辑要不要先分后合？ 

（2）程序如何分配运算任务（切片）？

（3）两阶段的程序如何启动？如何协调？

（4）整个程序运行过程中的监控？容错？重试？

分布式方案需要考虑很多问题，但是我们可以将分布式程序中的公共功能封装成框架，让开发人员将精力集中于业务逻辑上。而mapreduce就是这样一个分布式程序的通用框架。

## 1.2 MapReduce核心思想

![img](D:\software\Typora\imgs\clip_image002-1544403281327.jpg)

上图简单的阐明了map和reduce的两个过程或者作用，虽然不够严谨，但是足以提供一个大概的认知，map过程是一个蔬菜到制成食物前的准备工作，reduce将准备好的材料合并进而制作出食物的过程

![img](D:\software\Typora\imgs\clip_image004-1544403281340.png)

1）分布式的运算程序往往需要分成至少2个阶段

2）第一个阶段的maptask并发实例，完全并行运行，互不相干

3）第二个阶段的reduce task并发实例互不相干，但是他们的数据依赖于上一个阶段的所有maptask并发实例的输出

4）MapReduce编程模型只能包含一个map阶段和一个reduce阶段，如果用户的业务逻辑非常复杂，那就只能多个mapreduce程序，串行运行

## 1.3 MapReduce进程

一个完整的mapreduce程序在分布式运行时有三类实例进程：

1）MrAppMaster：负责整个程序的过程调度及状态协调

2）MapTask：负责map阶段的整个数据处理流程

3）ReduceTask：负责reduce阶段的整个数据处理流程

## 1.4 MapReduce编程规范（八股文）

 

用户编写的程序分成三个部分：Mapper，Reducer，Driver(提交运行mr程序的客户端)

1）Mapper阶段

​       （1）用户自定义的Mapper要继承自己的父类

​       （2）Mapper的输入数据是KV对的形式（KV的类型可自定义）

​       （3）Mapper中的业务逻辑写在map()方法中

​       （4）Mapper的输出数据是KV对的形式（KV的类型可自定义）

​       （5）map()方法（maptask进程）对每一个<K,V>调用一次

2）Reducer阶段

​       （1）用户自定义的Reducer要继承自己的父类

​       （2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV

​       （3）Reducer的业务逻辑写在reduce()方法中

​       （4）Reducetask进程对每一组相同k的<k,v>组调用一次reduce()方法

3）Driver阶段

整个程序需要一个Drvier来进行提交，提交的是一个描述了各种必要信息的job对象

4）案例实操

​       详见3.1.1统计一堆文件中单词出现的个数（WordCount案例）。

## 1.5 MapReduce程序运行流程分析

![img](D:\software\Typora\imgs\clip_image006-1544403281340.png)

1）在MapReduce程序读取文件的输入目录上存放相应的文件。

2）客户端程序在submit()方法执行前，获取待处理的数据信息，然后根据集群中参数的配置形成一个任务分配规划。

3）客户端提交job.split、jar包、job.xml等文件给yarn，yarn中的resourcemanager启动MRAppMaster。

4）MRAppMaster启动后根据本次job的描述信息，计算出需要的maptask实例数量，然后向集群申请机器启动相应数量的maptask进程。

5）maptask利用客户指定的inputformat来读取数据，形成输入KV对。

6）maptask将输入KV对传递给客户定义的map()方法，做逻辑运算

7）map()运算完毕后将KV对收集到maptask缓存。

8）maptask缓存中的KV对按照K分区排序后不断写到磁盘文件

9）MRAppMaster监控到所有maptask进程任务完成之后，会根据客户指定的参数启动相应数量的reducetask进程，并告知reducetask进程要处理的数据分区。

10）Reducetask进程启动之后，根据MRAppMaster告知的待处理数据所在位置，从若干台maptask运行所在机器上获取到若干个maptask输出结果文件，并在本地进行重新归并排序，然后按照相同key的KV为一个组，调用客户定义的reduce()方法进行逻辑运算。

11）Reducetask运算完毕后，调用客户指定的outputformat将结果数据输出到外部存储。

# 二 MapReduce理论篇

## 2.1 Writable序列化

序列化就是把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。 

反序列化就是将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。

Java的序列化是一个重量级序列化框架（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，header，继承体系等），不便于在网络中高效传输。所以，hadoop自己开发了一套序列化机制（Writable），精简、高效。

### 2.1.1 常用数据序列化类型

常用的数据类型对应的hadoop数据序列化类型

| **Java****类型** | **Hadoop   Writable****类型** |
| ---------------- | ----------------------------- |
| boolean          | BooleanWritable               |
| byte             | ByteWritable                  |
| int              | IntWritable                   |
| float            | FloatWritable                 |
| long             | LongWritable                  |
| double           | DoubleWritable                |
| string           | Text                          |
| map              | MapWritable                   |
| array            | ArrayWritable                 |

### 2.1.2 自定义bean对象实现序列化接口

1）自定义bean对象要想序列化传输，必须实现序列化接口，需要注意以下7项。

（1）必须实现Writable接口

（2）反序列化时，需要反射调用空参构造函数，所以必须有空参构造

（3）重写序列化方法

（4）重写反序列化方法

（5）注意反序列化的顺序和序列化的顺序完全一致

（6）要想把结果显示在文件中，需要重写toString()，且用”\t”分开，方便后续用

（7）如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序

   // 1 必须实现Writable接口   **public** **class** FlowBean **implements**   Writable {              **private** **long** upFlow;          **private** **long** downFlow;          **private** **long** sumFlow;              //2 反序列化时，需要反射调用空参构造函数，所以必须有          **public** FlowBean() {                 **super**();          }              /**           * 3重写序列化方法           *            * **@param** out           * **@throws** IOException           */          @Override          **public** **void** write(DataOutput   out) **throws** IOException {                 out.writeLong(upFlow);                 out.writeLong(downFlow);                 out.writeLong(sumFlow);          }              /**           * 4   重写反序列化方法    5 注意反序列化的顺序和序列化的顺序完全一致           *            * **@param** in           * **@throws** IOException           */          @Override          **public** **void**   readFields(DataInput in) **throws** IOException {                 upFlow = in.readLong();                 downFlow = in.readLong();                 sumFlow = in.readLong();          }           // 6要想把结果显示在文件中，需要重写toString()，且用”\t”分开，方便后续用          @Override          **public** String toString() {                 **return** upFlow +   "\t" + downFlow + "\t" + sumFlow;          }           //7 如果需要将自定义的bean放在key中传输，则还需要实现comparable接口，因为mapreduce框中的shuffle过程一定会对key进行排序          @Override          **public** **int**   compareTo(FlowBean o) {                 // 倒序排列，从大到小                 **return** **this**.sumFlow   > o.getSumFlow() ? -1 : 1;          }   }   

2）案例实操

​       详见3.2.1统计每一个手机号耗费的总上行流量、下行流量、总流量（序列化）。

## 2.2 InputFormat数据切片机制

### 2.2.1 FileInputFormat切片机制

1）job提交流程源码详解

waitForCompletion()

submit();

// 1建立连接

​       connect();       

​              // 1）创建提交job的代理

​              new Cluster(getConfiguration());

​                     // （1）判断是本地yarn还是远程

​                     initialize(jobTrackAddr, conf); 

​       // 2 提交job

submitter.submitJobInternal(Job.this, cluster)

​       // 1）创建给集群提交数据的*Stag*路径

​       Path jobStagingArea = JobSubmissionFiles.*getStagingDir*(cluster, conf);

​       // 2）获取jobid ，并创建job路径

​       JobID jobId = submitClient.getNewJobID();

​       // 3）拷贝jar包到集群

copyAndConfigureFiles(job, submitJobDir); 

​       rUploader.uploadFiles(job, jobSubmitDir);

// 4）计算切片，生成切片规划文件

writeSplits(job, submitJobDir);

​       maps = writeNewSplits(job, jobSubmitDir);

​              input.getSplits(job);

// 5）向*Stag**路径写xml**配置文件*

writeConf(conf, submitJobFile);

​       conf.writeXml(out);

// 6）提交job,返回提交状态

status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());

![img](D:\software\Typora\imgs\clip_image008-1544403281340.png)2）FileInputFormat源码解析(input.getSplits(job))

（1）找到你数据存储的目录。

​       （2）开始遍历处理（规划切片）目录下的每一个文件

​       （3）遍历第一个文件ss.txt

​              a）获取文件大小fs.sizeOf(ss.txt);

​              b）计算切片大小computeSliteSize(Math.max(minSize,Math.max(maxSize,blocksize)))=blocksize=128M

c）默认情况下，切片大小=blocksize

​              d）开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M（每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片）

​              e）将切片信息写到一个切片规划文件中

​              f）整个切片的核心过程在getSplit()方法中完成。

g）数据切片只是在逻辑上对输入数据进行分片，并不会再磁盘上将其切分成分片进行存储。InputSplit只记录了分片的元数据信息，比如起始位置、长度以及所在的节点列表等。

h）注意：block是HDFS上物理上存储的存储的数据，切片是对数据逻辑上的划分。

​       （4）提交切片规划文件到yarn上，yarn上的MrAppMaster就可以根据切片规划文件计算开启maptask个数。

3）FileInputFormat中默认的切片机制：

（1）简单地按照文件的内容长度进行切片

（2）切片大小，默认等于block大小

（3）切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

比如待处理数据有两个文件：



   file1.txt      320M   file2.txt      10M   

经过FileInputFormat的切片机制运算后，形成的切片信息如下：  



   file1.txt.split1--  0~128   file1.txt.split2--  128~256   file1.txt.split3--  256~320   file2.txt.split1--  0~10M   

4）FileInputFormat切片大小的参数配置

（1）通过分析源码，在FileInputFormat中，计算切片大小的逻辑：Math.max(minSize, Math.min(maxSize, blockSize));  

切片主要由这几个值来运算决定

mapreduce.input.fileinputformat.split.minsize=1 默认值为1

mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue 默认值Long.MAXValue

因此，默认情况下，切片大小=blocksize。

maxsize（切片最大值）：参数如果调得比blocksize小，则会让切片变小，而且就等于配置的这个参数的值。

minsize （切片最小值）：参数调的比blockSize大，则可以让切片变得比blocksize还大。

5）获取切片信息API



   // 根据文件类型获取切片信息   FileSplit inputSplit = (FileSplit) context.getInputSplit();   // 获取切片的文件名称   String name = inputSplit.getPath().getName();   

### 2.2.2 CombineTextInputFormat切片机制

关于大量小文件的优化策略

1）默认情况下TextInputformat对任务的切片机制是按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个maptask，这样如果有大量小文件，就会产生大量的maptask，处理效率极其低下。

2）优化策略

​       （1）最好的办法，在数据处理系统的最前端（预处理/采集），将小文件先合并成大文件，再上传到HDFS做后续分析。

​       （2）补救措施：如果已经是大量小文件在HDFS中了，可以使用另一种InputFormat来做切片（CombineTextInputFormat），它的切片逻辑跟TextFileInputFormat不同：它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个maptask。

​       （3）优先满足最小切片大小，不超过最大切片大小

​              CombineTextInputFormat.*setMaxInputSplitSize*(job, 4194304);// 4m

​              CombineTextInputFormat.*setMinInputSplitSize*(job, 2097152);// 2m

​       举例：0.5m+1m+0.3m+5m=2m + 4.8m=2m + 4m + 0.8m

3）具体实现步骤



   // 9 如果不设置InputFormat,它默认用的是TextInputFormat.class   job.setInputFormatClass(CombineTextInputFormat.**class**)   CombineTextInputFormat.*setMaxInputSplitSize*(job, 4194304);//   4m   CombineTextInputFormat.*setMinInputSplitSize*(job, 2097152);//   2m   

4）案例实操

​       详见3.1.4 需求4：大量小文件的切片优化（CombineTextInputFormat）。

### 2.2.3 自定义InputFormat

1）概述

（1）自定义一个InputFormat

（2）改写RecordReader，实现一次读取一个完整文件封装为KV

（3）在输出时使用SequenceFileOutPutFormat输出合并文件

2）案例实操

​       详见3.5小文件处理（自定义InputFormat）。

## 2.3 MapTask工作机制 

1）问题引出

maptask的并行度决定map阶段的任务处理并发度，进而影响到整个job的处理速度。那么，mapTask并行任务是否越多越好呢？ 

2）MapTask并行度决定机制

​       一个job的map阶段MapTask并行度（个数），由客户端提交job时的切片个数决定。

3）MapTask工作机制

​       （1）Read阶段：Map Task通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。

​       （2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。

​       （3）Collect阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。

​       （4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

​       溢写阶段详情：

​       步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。

​       步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。

​       步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当期内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。

​       （5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

​       当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。

​       在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认100）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。

​       让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。











## 2.4 Shuffle机制

### 2.4.1 Shuffle机制

Mapreduce确保每个reducer的输入都是按键排序的。系统执行排序的过程（即将map输出作为输入传给reducer）称为shuffle。

![4df193f5-e56e-308f-9689-eac035dd8a2b](D:\software\Typora\imgs\clip_image010-1544403281340.jpg)

### 2.4.2 MapReduce工作流程

1）流程示意图

![img](D:\software\Typora\imgs\clip_image012-1544403281340.png)

![img](D:\software\Typora\imgs\clip_image014-1544403281340.png)

2）流程详解

上面的流程是整个mapreduce最全工作流程，但是shuffle过程只是从第7步开始到第16步结束，具体shuffle过程详解，如下：

1）maptask收集我们的map()方法输出的kv对，放到内存缓冲区中

2）从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件

3）多个溢出文件会被合并成大的溢出文件

4）在溢出过程中，及合并的过程中，都要调用partitoner进行分组和针对key进行排序

5）reducetask根据自己的分区号，去各个maptask机器上取相应的结果分区数据

6）reducetask会取到同一个分区的来自不同maptask的结果文件，reducetask会将这些文件再进行合并（归并排序）

7）合并成大文件后，shuffle的过程也就结束了，后面进入reducetask的逻辑运算过程（从文件中取出一个一个的键值对group，调用用户自定义的reduce()方法）

3）注意

Shuffle中的缓冲区大小会影响到mapreduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。

缓冲区的大小可以通过参数调整，参数：io.sort.mb  默认100M

### 2.4.3 partition分区

0）问题引出：要求将统计结果按照条件输出到不同文件中（分区）。比如：将统计结果按照手机归属地不同省份输出到不同文件中（分区）

1）默认partition分区

   public class HashPartitioner<K, V> extends Partitioner<K,   V> {     /** Use {@link   Object#hashCode()} to partition. */     public int getPartition(K key,   V value, int numReduceTasks) {       return (key.hashCode() &   Integer.MAX_VALUE) % numReduceTasks;     }   }   

​       默认分区是根据key的hashCode对reduceTasks个数取模得到的。用户没法控制哪个key存储到哪个分区

2）自定义Partitioner步骤

​       （1）自定义类继承Partitioner，重新getPartition()方法

​          **public** **class**   ProvincePartitioner **extends** Partitioner<Text, FlowBean> {          @Override          **public** **int**   getPartition(Text key, FlowBean value, **int** numPartitions) {   // 1 获取电话号码的前三位                 String preNum =   key.toString().substring(0, 3);                                  **int** partition = 4;                                  // 2 判断是哪个省                 **if**   ("136".equals(preNum)) {                        partition = 0;                 }**else** **if**   ("137".equals(preNum)) {                        partition = 1;                 }**else** **if**   ("138".equals(preNum)) {                        partition = 2;                 }**else** **if**   ("139".equals(preNum)) {                        partition = 3;                 }                 **return** partition;          }   }   

​       （2）在job驱动中，设置自定义partitioner： 

   job.setPartitionerClass(CustomPartitioner.class)   

​       （3）自定义partition后，要根据自定义partitioner的逻辑设置相应数量的reduce task

   job.setNumReduceTasks(5);   

3）注意：

如果reduceTask的数量> getPartition的结果数，则会多产生几个空的输出文件part-r-000xx；

如果1<reduceTask的数量<getPartition的结果数，则有一部分分区数据无处安放，会Exception；

如果reduceTask的数量=1，则不管mapTask端输出多少个分区文件，最终结果都交给这一个reduceTask，最终也就只会产生一个结果文件 part-r-00000；

​       例如：假设自定义分区数为5，则

（1）job.setNumReduceTasks(1);会正常运行，只不过会产生一个输出文件

（2）job.setNumReduceTasks(2);会报错

（3）job.setNumReduceTasks(6);大于5，程序会正常运行，会产生空文件

4）案例实操

详见3.2.2 需求2：将统计结果按照手机归属地不同省份输出到不同文件中（Partitioner）

详见3.1.2 需求2：把单词按照ASCII码奇偶分区（Partitioner）

### 2.4.4 排序

排序是MapReduce框架中最重要的操作之一。Map Task和Reduce Task均会对数据（按照key）进行排序。该操作属于Hadoop的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。

​       对于Map Task，它会将处理的结果暂时放到一个缓冲区中，当缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次排序，并将这些有序数据写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行一次合并，以将这些文件合并成一个大的有序文件。

​       对于Reduce Task，它从每个Map Task上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则放到磁盘上，否则放到内存中。如果磁盘上文件数目达到一定阈值，则进行一次合并以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据写到磁盘上。当所有数据拷贝完毕后，Reduce Task统一对内存和磁盘上的所有数据进行一次合并。

每个阶段的默认排序

1）排序的分类：

​       （1）部分排序：

MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部排序。

​       （2）全排序：

如何用Hadoop产生一个全局排序的文件？最简单的方法是使用一个分区。但该方法在处理大型文件时效率极低，因为一台机器必须处理所有输出文件，从而完全丧失了MapReduce所提供的并行架构。

​       替代方案：首先创建一系列排好序的文件；其次，串联这些文件；最后，生成一个全局排序的文件。主要思路是使用一个分区来描述输出的全局排序。例如：可以为上述文件创建3个分区，在第一分区中，记录的单词首字母a-g，第二分区记录单词首字母h-n, 第三分区记录单词首字母o-z。

（3）辅助排序：（GroupingComparator分组）

​       Mapreduce框架在记录到达reducer之前按键对记录排序，但键所对应的值并没有被排序。甚至在不同的执行轮次中，这些值的排序也不固定，因为它们来自不同的map任务且这些map任务在不同轮次中完成时间各不相同。一般来说，大多数MapReduce程序会避免让reduce函数依赖于值的排序。但是，有时也需要通过特定的方法对键进行排序和分组等以实现对值的排序。

 

2）自定义排序WritableComparable

（1）原理分析

bean对象实现WritableComparable接口重写compareTo方法，就可以实现排序

   @Override   **public** **int** compareTo(FlowBean o) {          // 倒序排列，从大到小          **return** **this**.sumFlow > o.getSumFlow() ? -1 :   1;   }   

（2）案例实操

详见3.2.3 需求3：将统计结果按照总流量倒序排序（排序）

### 2.4.5 GroupingComparator分组

1）对reduce阶段的数据根据某一个或几个字段进行分组。

2）案例实操

详见3.3 求出每一个订单中最贵的商品（GroupingComparator）

### 2.4.6 Combiner合并

1）combiner是MR程序中Mapper和Reducer之外的一种组件

2）combiner组件的父类就是Reducer

3）combiner和reducer的区别在于运行的位置：

Combiner是在每一个maptask所在的节点运行

Reducer是接收全局所有Mapper的输出结果；

4）combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量

5）自定义Combiner实现步骤：

（1）自定义一个combiner继承Reducer，重写reduce方法

   public class   WordcountCombiner extends Reducer<Text, IntWritable, Text,   IntWritable>{          @Override          protected void reduce(Text key,   Iterable<IntWritable> values,                        Context context) throws   IOException, InterruptedException {                 int count = 0;                 for(IntWritable v :values){                        count = v.get();                 }                 context.write(key, new   IntWritable(count));          }   }   

（2）在job中设置：  

   job.setCombinerClass(WordcountCombiner.class);   

6）combiner能够应用的前提是不能影响最终的业务逻辑，而且，combiner的输出kv应该跟reducer的输入kv类型要对应起来

Mapper

3 5 7 ->(3+5+7)/3=5 

2 6 ->(2+6)/2=4

Reducer

(3+5+7+2+6)/5=23/5    不等于    (5+4)/2=9/2

7）案例实操

详见3.1.3需求3：对每一个maptask的输出局部汇总（Combiner）

### 2.4.7 数据倾斜&Distributedcache

1）数据倾斜原因

如果是多张表的操作都是在reduce阶段完成，reduce端的处理压力太大，map节点的运算负载则很低，资源利用率不高，且在reduce阶段极易产生数据倾斜。

2）实操案例：

详见3.4.1 需求1：reduce端表合并（数据倾斜）

3）解决方案

在map端缓存多张表，提前处理业务逻辑，这样增加map端业务，减少reduce端数据的压力，尽可能的减少数据倾斜。

4）具体办法：采用distributedcache

​       （1）在mapper的setup阶段，将文件读取到缓存集合中

​       （2）在驱动函数中加载缓存。

job.addCacheFile(new URI("file:/e:/mapjoincache/pd.txt"));// 缓存普通文件到task运行节点

5）实操案例：

详见3.4.2需求2：map端表合并（Distributedcache）

## 2.5 ReduceTask工作机制

1）设置ReduceTask

reducetask的并行度同样影响整个job的执行并发度和执行效率，但与maptask的并发数由切片数决定不同，Reducetask数量的决定是可以直接手动设置：

   //默认值是1，手动设置为4   job.setNumReduceTasks(4);   

2）注意

（1）如果数据分布不均匀，就有可能在reduce阶段产生数据倾斜

（2）reducetask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个reducetask。

（3）具体多少个reducetask，需要根据集群性能而定。

（4）如果分区数不是1，但是reducetask为1，是否执行分区过程。答案是：不执行分区过程。因为在maptask的源码中，执行分区的前提是先判断reduceNum个数是否大于1。不大于1肯定不执行。

3）实验：测试reducetask多少合适。

（1）实验环境：1个master节点，16个slave节点: CPU:8GHZ , 内存: 2G

（2）实验结论：

表1 改变reduce task （数据量为1GB）

| Map task =16 |      |      |      |      |      |      |      |      |      |      |
| ------------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Reduce task  | 1    | 5    | 10   | 15   | 16   | 20   | 25   | 30   | 45   | 60   |
| 总时间       | 892  | 146  | 110  | 92   | 88   | 100  | 128  | 101  | 145  | 104  |

4）ReduceTask工作机制

​       （1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。

​       （2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。

​       （3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。

​       （4）Reduce阶段：reduce()函数将计算结果写到HDFS上。

## 2.6 自定义OutputFormat

1）概述

（1）要在一个mapreduce程序中根据数据的不同输出两类结果到不同目录，这类灵活的输出需求可以通过自定义outputformat来实现。

（1）自定义outputformat，

（2）改写recordwriter，具体改写输出数据的方法write()

2）实操案例：

详见3.6 修改日志内容及自定义日志输出路径（自定义OutputFormat）。

## 2.7 MapReduce数据压缩

### 2.7.1 概述

压缩技术能够有效减少底层存储系统（HDFS）读写字节数。压缩提高了网络带宽和磁盘空间的效率。在Hadood下，尤其是数据规模很大和工作负载密集的情况下，使用数据压缩显得非常重要。在这种情况下，I/O操作和网络数据传输要花大量的时间。还有，Shuffle与Merge过程同样也面临着巨大的I/O压力。

​       鉴于磁盘I/O和网络带宽是Hadoop的宝贵资源，数据压缩对于节省资源、最小化磁盘I/O和网络传输非常有帮助。不过，尽管压缩与解压操作的CPU开销不高，其性能的提升和资源的节省并非没有代价。

​       如果磁盘I/O和网络带宽影响了MapReduce作业性能，在任意MapReduce阶段启用压缩都可以改善端到端处理时间并减少I/O和网络流量。

压缩**mapreduce****的一种优化策略：通过压缩编码对****mapper****或者****reducer****的输出进行压缩，以减少磁盘****IO****，**提高MR程序运行速度（但相应增加了cpu运算负担）

注意：压缩特性运用得当能提高性能，但运用不当也可能降低性能

基本原则：

（1）运算密集型的job，少用压缩

（2）IO密集型的job，多用压缩

### 2.7.2 MR支持的压缩编码

| 压缩格式 | 工具  | 算法    | 文件扩展名 | 是否可切分 |
| -------- | ----- | ------- | ---------- | ---------- |
| DEFAULT  | 无    | DEFAULT | .deflate   | 否         |
| Gzip     | gzip  | DEFAULT | .gz        | 否         |
| bzip2    | bzip2 | bzip2   | .bz2       | 是         |
| LZO      | lzop  | LZO     | .lzo       | 否         |
| LZ4      | 无    | LZ4     | .lz4       | 否         |
| Snappy   | 无    | Snappy  | .snappy    | 否         |

为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示

| 压缩格式 | 对应的编码/解码器                          |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| LZ4      | org.apache.hadoop.io.compress.Lz4Codec     |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

压缩性能的比较

| 压缩算法 | 原始文件大小 | 压缩文件大小 | 压缩速度 | 解压速度 |
| -------- | ------------ | ------------ | -------- | -------- |
| gzip     | 8.3GB        | 1.8GB        | 17.5MB/s | 58MB/s   |
| bzip2    | 8.3GB        | 1.1GB        | 2.4MB/s  | 9.5MB/s  |
| LZO-bset | 8.3GB        | 2GB          | 4MB/s    | 60.6MB/s |
| LZO      | 8.3GB        | 2.9GB        | 49.3MB/s | 74.6MB/s |

 

### 2.7.3 采用压缩的位置

​       压缩可以在MapReduce作用的任意阶段启用。

![img](D:\software\Typora\imgs\clip_image016-1544403281340.png)

1）输入压缩：

​       在有大量数据并计划重复处理的情况下，应该考虑对输入进行压缩。然而，你无须显示指定使用的编解码方式。Hadoop自动检查文件扩展名，如果扩展名能够匹配，就会用恰当的编解码方式对文件进行压缩和解压。否则，Hadoop就不会使用任何编解码器。

2）压缩mapper输出：

当map任务输出的中间数据量很大时，应考虑在此阶段采用压缩技术。这能显著改善内部数据Shuffle过程，而Shuffle过程在Hadoop处理过程中是资源消耗最多的环节。如果发现数据量大造成网络传输缓慢，应该考虑使用压缩技术。可用于压缩mapper输出的快速编解码器包括LZO、LZ4或者Snappy。

​       注：LZO是供Hadoop压缩数据用的通用压缩编解码器。其设计目标是达到与硬盘读取速度相当的压缩速度，因此速度是优先考虑的因素，而不是压缩率。与gzip编解码器相比，它的压缩速度是gzip的5倍，而解压速度是gzip的2倍。同一个文件用LZO压缩后比用gzip压缩后大50%，但比压缩前小25%~50%。这对改善性能非常有利，map阶段完成时间快4倍。

3）压缩reducer输出：

​       在此阶段启用压缩技术能够减少要存储的数据量，因此降低所需的磁盘空间。当mapreduce作业形成作业链条时，因为第二个作业的输入也已压缩，所以启用压缩同样有效。

### 2.7.4 压缩配置参数

要在Hadoop中启用压缩，可以配置如下参数（mapred-site.xml文件中）：

| 参数                                                 | 默认值                                                       | 阶段        | 建议                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ | ----------- | -------------------------------------------- |
| io.compression.codecs      （在core-site.xml中配置） | org.apache.hadoop.io.compress.DefaultCodec,   org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec,   org.apache.hadoop.io.compress.Lz4Codec | 输入压缩    | Hadoop使用文件扩展名判断是否支持某种编解码器 |
| mapreduce.map.output.compress                        | false                                                        | mapper输出  | 这个参数设为true启用压缩                     |
| mapreduce.map.output.compress.codec                  | org.apache.hadoop.io.compress.DefaultCodec                   | mapper输出  | 使用LZO、LZ4或snappy编解码器在此阶段压缩数据 |
| mapreduce.output.fileoutputformat.compress           | false                                                        | reducer输出 | 这个参数设为true启用压缩                     |
| mapreduce.output.fileoutputformat.compress.codec     | org.apache.hadoop.io.compress. DefaultCodec                  | reducer输出 | 使用标准工具或者编解码器，如gzip和bzip2      |
| mapreduce.output.fileoutputformat.compress.type      | RECORD                                                       | reducer输出 | SequenceFile输出使用的压缩类型：NONE和BLOCK  |

 

## 2.8 计数器应用

​       Hadoop为每个作业维护若干内置计数器，以描述多项指标。例如，某些计数器记录已处理的字节数和记录数，使用户可监控已处理的输入数据量和已产生的输出数据量。

1）API

​       （1）采用枚举的方式统计计数

enum MyCounter{MALFORORMED,NORMAL}

//对枚举定义的自定义计数器加1

context.getCounter(MyCounter.MALFORORMED).increment(1);

（2）采用计数器组、计数器名称的方式统计

context.getCounter("counterGroup", "countera").increment(1);

​              组名和计数器名称随便起，但最好有意义。

​       （3）计数结果在程序运行后的控制台上查看。

2）案例实操

详见3.6 修改日志内容及自定义日志输出路径（自定义OutputFormat）。

## 2.9 数据清洗

1）概述

在运行Mapreduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行mapper程序，不需要运行reduce程序。

2）实操案例

详见3.7 日志清洗（数据清洗）。

## 2.10 MapReduce与Yarn

### 2.10.1 Yarn概述

Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而mapreduce等运算程序则相当于运行于操作系统之上的应用程序

### 2.10.2 Yarn的重要概念

1）Yarn并不清楚用户提交的程序的运行机制

2）Yarn只提供运算资源的调度（用户程序向Yarn申请资源，Yarn就负责分配资源）

3）Yarn中的主管角色叫ResourceManager

4）Yarn中具体提供运算资源的角色叫NodeManager

5）这样一来，Yarn其实就与运行的用户程序完全解耦，就意味着Yarn上可以运行各种类型的分布式运算程序（mapreduce只是其中的一种），比如mapreduce、storm程序，spark程序……

6）所以，spark、storm等运算框架都可以整合在Yarn上运行，只要他们各自的框架中有符合Yarn规范的资源请求机制即可

7）Yarn就成为一个通用的资源调度平台，从此，企业中以前存在的各种运算集群都可以整合在一个物理集群上，提高资源利用率，方便数据共享

### 2.10.3 Yarn工作机制

1）Yarn运行机制

![img](D:\software\Typora\imgs\clip_image018-1544403281340.png)

2）工作机制详解

​       （0）Mr程序提交到客户端所在的节点

​       （1）yarnrunner向Resourcemanager申请一个application。

​       （2）rm将该应用程序的资源路径返回给yarnrunner

​       （3）该程序将运行所需资源提交到HDFS上

​       （4）程序资源提交完毕后，申请运行mrAppMaster

​       （5）RM将用户的请求初始化成一个task

​       （6）其中一个NodeManager领取到task任务。

​       （7）该NodeManager创建容器Container，并产生MRAppmaster

​       （8）Container从HDFS上拷贝资源到本地

​       （9）MRAppmaster向RM 申请运行maptask容器

​       （10）RM将运行maptask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

​       （11）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动maptask，maptask对数据分区排序。

​       （12）MRAppmaster向RM申请2个容器，运行reduce task。

​       （13）reduce task向maptask获取相应分区的数据。

​       （14）程序运行完毕后，MR会向RM注销自己。

## 2.11 作业提交全过程

1）作业提交过程之YARN

![img](D:\software\Typora\imgs\clip_image020-1544403281340.png)

2）作业提交过程之MapReduce

![img](D:\software\Typora\imgs\clip_image022-1544403281341.png)

3）作业提交过程之读数据

![img](D:\software\Typora\imgs\clip_image024-1544403281341.png)

4）作业提交过程之写数据

![img](D:\software\Typora\imgs\clip_image026-1544403281341.png)

## 2.12 MapReduce开发总结

mapreduce在编程的时候，基本上一个固化的模式，没有太多可灵活改变的地方，除了以下几处：

**1****）输入数据接口**：InputFormat--->FileInputFormat(文件类型数据读取的通用抽象类)  DBInputFormat （数据库数据读取的通用抽象类）

   默认使用的实现类是：TextInputFormat 

   job.setInputFormatClass(TextInputFormat.class)

   TextInputFormat的功能逻辑是：一次读一行文本，然后将该行的起始偏移量作为key，行内容作为value返回

**2****）逻辑处理接口**： Mapper  

   完全需要用户自己去实现其中：map()   setup()   clean() 

3）map输出的结果在shuffle阶段会被partition以及sort，此处有两个接口可自定义：

​    **（1****）Partitioner**

​       有默认实现 HashPartitioner，逻辑是  根据key和numReduces来返回一个分区号； key.hashCode()&Integer.MAXVALUE % numReduces

​       通常情况下，用默认的这个HashPartitioner就可以，如果业务上有特别的需求，可以自定义

​       **（2****）Comparable**

​       当我们用自定义的对象作为key来输出时，就必须要实现WritableComparable接口，override其中的compareTo()方法

**4****）reduce****端的数据分组比较接口：**Groupingcomparator

​       reduceTask拿到输入数据（一个partition的所有数据）后，首先需要对数据进行分组，其分组的默认原则是key相同，然后对每一组kv数据调用一次reduce()方法，并且将这一组kv中的第一个kv的key作为参数传给reduce的key，将这一组数据的value的迭代器传给reduce()的values参数

​       利用上述这个机制，我们可以实现一个高效的分组取最大值的逻辑：

​       自定义一个bean对象用来封装我们的数据，然后改写其compareTo方法产生倒序排序的效果

​       然后自定义一个Groupingcomparator，将bean对象的分组逻辑改成按照我们的业务分组id来分组（比如订单号）

​       这样，我们要取的最大值就是reduce()方法中传进来key

**5****）逻辑处理接口：**Reducer

​       完全需要用户自己去实现其中  reduce()   setup()   clean()   

**6****）输出数据接口：**OutputFormat---> 有一系列子类FileOutputformat  DBoutputFormat  .....

​       默认实现类是TextOutputFormat，功能逻辑是：将每一个KV对向目标文本文件中输出为一行

## 2.13 MapReduce参数优化

### 2.13.1 资源相关参数

1）以下参数是在用户自己的mr应用程序中配置就可以生效

| 配置参数                    | 参数说明                                                     |
| --------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb     | 一个Map Task可使用的资源上限（单位:MB），默认为1024。如果Map Task实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.reduce.memory.mb  | 一个Reduce   Task可使用的资源上限（单位:MB），默认为1024。如果Reduce Task实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.java.opts     | Map Task的JVM参数，你可以在此配置默认的java heap size等参数, e.g.   “-Xmx1024m -verbose:gc -Xloggc:/tmp/@taskid@.gc” （@taskid@会被Hadoop框架自动换为相应的taskid）, 默认值: “” |
| mapreduce.reduce.java.opts  | Reduce Task的JVM参数，你可以在此配置默认的java heap size等参数, e.g.   “-Xmx1024m   -verbose:gc -Xloggc:/tmp/@taskid@.gc”, 默认值: “” |
| mapreduce.map.cpu.vcores    | 每个Map task可使用的最多cpu core数目, 默认值: 1              |
| mapreduce.reduce.cpu.vcores | 每个Reduce task可使用的最多cpu core数目, 默认值: 1           |

2）应该在yarn启动之前就配置在服务器的配置文件中才能生效

| 配置参数                                     | 参数说明                          |
| -------------------------------------------- | --------------------------------- |
| yarn.scheduler.minimum-allocation-mb   1024  | 给应用程序container分配的最小内存 |
| yarn.scheduler.maximum-allocation-mb   8192  | 给应用程序container分配的最大内存 |
| yarn.scheduler.minimum-allocation-vcores   1 |                                   |
| yarn.scheduler.maximum-allocation-vcores  32 |                                   |
| yarn.nodemanager.resource.memory-mb   8192   |                                   |

3）shuffle性能优化的关键参数，应在yarn启动之前就配置好

| 配置参数                               | 参数说明                          |
| -------------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb   100        | shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent   0.8 | 环形缓冲区溢出的阈值，默认80%     |

### 2.13.2 容错相关参数

| 配置参数                             | 参数说明                                                     |
| ------------------------------------ | ------------------------------------------------------------ |
| mapreduce.map.maxattempts            | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts         | 每个Reduce   Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.map.failures.maxpercent    | 当失败的Map Task失败比例超过该值为，整个作业则失败，默认值为0. 如果你的应用程序允许丢弃部分输入数据，则该该值设为一个大于0的值，比如5，表示如果有低于5%的Map Task失败（如果一个Map Task重试次数超过mapreduce.map.maxattempts，则认为这个Map Task失败，其对应的输入数据将不会产生任何结果），整个作业扔认为成功。 |
| mapreduce.reduce.failures.maxpercent | 当失败的Reduce   Task失败比例超过该值为，整个作业则失败，默认值为0。 |
| mapreduce.task.timeout               | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该task处于block状态，可能是卡住了，也许永远会卡主，为了防止因为用户程序永远block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是300000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after   300 secsContainer killed by the ApplicationMaster.”。 |

# 三 MapReduce实战篇

## 3.1 WordCount案例

### 3.1.1 需求1：统计一堆文件中单词出现的个数（WordCount案例）

0）需求：在一堆给定的文本文件中统计输出每一个单词出现的总次数

1）数据准备：

![img](D:\software\Typora\imgs\clip_image028-1544403281341.png)

2）分析

​       按照mapreduce编程规范，分别编写Mapper，Reducer，Driver。

![img](D:\software\Typora\imgs\clip_image030-1544403281341.png)

![img](D:\software\Typora\imgs\clip_image032-1544403281341.png)

3）编写程序

（1）定义一个mapper类

   **package** com.atguigu.wordcount;       **import** java.io.IOException;       **import** org.apache.hadoop.io.IntWritable;   **import** org.apache.hadoop.io.LongWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Mapper;       /**    * KEYIN:默认情况下，是mr框架所读到的一行文本的起始偏移量，Long;    * 在hadoop中有自己的更精简的序列化接口，所以不直接用Long，而是用LongWritable    * VALUEIN:默认情况下，是mr框架所读到的一行文本内容，String;此处用Text    *     * KEYOUT:是用户自定义逻辑处理完成之后输出数据中的key,在此处是单词，String；此处用Text    * VALUEOUT，是用户自定义逻辑处理完成之后输出数据中的value，在此处是单词次数，Integer，此处用IntWritable    * **@author**   Administrator    */   **public** **class** WordcountMapper **extends**   Mapper<LongWritable, Text, Text, IntWritable>{          /**           * map阶段的业务逻辑就写在自定义的map()方法中           * maptask会对每一行输入数据调用一次我们自定义的map（）方法           */          @Override          **protected** **void**   map(LongWritable key, Text value, Context context)                        **throws**   IOException, InterruptedException {                 // 1 将maptask传给我们的文本内容先转换成String                 String line =   value.toString();                                  // 2 根据空格将这一行切分成单词                 String[]   words = line.split(" ");                                  // 3 将单词输出为<单词，1>                 **for**(String   word:words){                        // 将单词作为key，将次数1作为value,以便于后续的数据分发，可以根据单词分发，以便于相同单词会到相同的reducetask中                        context.write(**new**   Text(word), **new** IntWritable(1));                 }          }   }   

（2）定义一个reducer类

   **package** com.atguigu.wordcount;   **import** java.io.IOException;   **import** org.apache.hadoop.io.IntWritable;   **import** org.apache.hadoop.io.LongWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Reducer;       /**    * KEYIN , VALUEIN 对应mapper输出的KEYOUT, VALUEOUT类型    *     * KEYOUT，VALUEOUT 对应自定义reduce逻辑处理结果的输出数据类型 KEYOUT是单词 VALUEOUT是总次数   * **@author** Administrator   */   **public** **class** WordcountReducer **extends**   Reducer<Text, IntWritable, Text, IntWritable> {              /**           * key，是一组相同单词kv对的key           */          @Override          **protected** **void**   reduce(Text key, Iterable<IntWritable> values, Context context)                        **throws**   IOException, InterruptedException {                     **int**   count = 0;                     // 1 汇总各个key的个数                 **for**(IntWritable   value:values){                        count   +=value.get();                 }                                  // 2输出该key的总次数                 context.write(key,   **new** IntWritable(count));          }   }   

（3）定义一个主类，用来描述job并提交job

   **package** com.atguigu.wordcount;   **import** java.io.IOException;   **import** org.apache.hadoop.conf.Configuration;   **import** org.apache.hadoop.fs.Path;   **import** org.apache.hadoop.io.IntWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Job;   **import** org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import** org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       /**    * 相当于一个yarn集群的客户端，    * 需要在此封装我们的mr程序相关运行参数，指定jar包    * 最后提交给yarn    * **@author**   Administrator   */   **public** **class** WordcountDriver {          **public** **static**   **void** main(String[] args) **throws** Exception {                 // 1 获取配置信息，或者job对象实例                 Configuration   configuration = **new** Configuration();                 // 8 配置提交到yarn上运行,windows和Linux变量不一致   //            configuration.set("mapreduce.framework.name",   "yarn");   //            configuration.set("yarn.resourcemanager.hostname",   "hadoop103");                 Job job =   Job.*getInstance*(configuration);                                  // 6 指定本程序的jar包所在的本地路径   //            job.setJar("/home/atguigu/wc.jar");                 job.setJarByClass(WordcountDriver.**class**);                                  // 2 指定本业务job要使用的mapper/Reducer业务类                 job.setMapperClass(WordcountMapper.**class**);                 job.setReducerClass(WordcountReducer.**class**);                                  // 3 指定mapper输出数据的kv类型                 job.setMapOutputKeyClass(Text.**class**);                 job.setMapOutputValueClass(IntWritable.**class**);                                  // 4 指定最终输出的数据的kv类型                 job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(IntWritable.**class**);                                  // 5 指定job的输入原始文件所在目录                 FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                                  // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行   //            job.submit();                 **boolean**   result = job.waitForCompletion(**true**);                 System.*exit*(result?0:1);          }   }   

（4）将程序打成jar包，然后拷贝到hadoop集群中。

（5）启动hadoop集群

（6）执行wordcount程序

[atguigu@hadoop102 software]$ hadoop jar  wc.jar com.atguigu.wordcount.WordcountDriver /user/atguigu/input /user/atguigu/output1

### 3.1.2 需求2：把单词按照ASCII码奇偶分区（Partitioner）

0）分析

![img](D:\software\Typora\imgs\clip_image034-1544403281341.png)

1）自定义分区

   **package**   com.atguigu.mapreduce.wordcount;   **import**   org.apache.hadoop.io.IntWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Partitioner;       **public** **class**   WordCountPartitioner **extends** Partitioner<Text, IntWritable>{              @Override          **public**   **int** getPartition(Text key, IntWritable value, **int**   numPartitions) {                                  //   1 获取单词key                   String   firWord = key.toString().substring(0, 1);                 **char**[]   charArray = firWord.toCharArray();                 **int**   result = charArray[0];                 //   **int** result  =   key.toString().charAt(0);                     //   2 根据奇数偶数分区                 **if**   (result % 2 == 0) {                        **return**   0;                 }**else**   {                        **return**   1;                 }          }   }   

2）在驱动中配置加载分区，设置reducetask个数

   job.setPartitionerClass(WordCountPartitioner.**class**);   job.setNumReduceTasks(2);   

### 3.1.3 需求3：对每一个maptask的输出局部汇总（Combiner）

0）需求：统计过程中对每一个maptask的输出进行局部汇总，以减小网络传输量即采用Combiner功能。

![img](D:\software\Typora\imgs\clip_image036-1544403281341.png)

1）数据准备：

![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image028.png)

**方案一**

1）增加一个WordcountCombiner类继承Reducer

   **package**   com.atguigu.mr.combiner;   **import**   java.io.IOException;   **import**   org.apache.hadoop.io.IntWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Reducer;       **public** **class**   WordcountCombiner **extends** Reducer<Text, IntWritable, Text, IntWritable>{              @Override          **protected**   **void** reduce(Text key, Iterable<IntWritable> values,                        Context   context) **throws** IOException, InterruptedException {                     **int**   count = 0;                 **for**(IntWritable   v :values){                        count   = v.get();                 }                                  context.write(key,   **new** IntWritable(count));          }   }   

2）在WordcountDriver驱动类中指定combiner

   // 9 指定需要使用combiner，以及用哪个类作为combiner的逻辑   job.setCombinerClass(WordcountCombiner.class);   

**方案二**

1）将WordcountReducer作为combiner在WordcountDriver驱动类中指定

   // 9 指定需要使用combiner，以及用哪个类作为combiner的逻辑   job.setCombinerClass(WordcountReducer.**class**);   

 

运行程序

![img](D:\software\Typora\imgs\clip_image039-1544403281341.jpg)

![img](D:\software\Typora\imgs\clip_image041-1544403281341.jpg)

### 3.1.4 需求4：大量小文件的切片优化（CombineTextInputFormat）

0）需求：将输入的大量小文件合并成一个切片统一处理。

1）输入数据：准备5个小文件

2）实现过程

（1）不做任何处理，运行需求1中的wordcount程序，观察切片个数为5

![img](D:\software\Typora\imgs\clip_image043-1544403281341.jpg)

（2）在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为1

   // 9 如果不设置InputFormat,它默认用的是TextInputFormat.class   job.setInputFormatClass(CombineTextInputFormat.**class**);   CombineTextInputFormat.*setMaxInputSplitSize*(job,   4194304);// 4m   CombineTextInputFormat.*setMinInputSplitSize*(job,   2097152);// 2m   

![img](D:\software\Typora\imgs\clip_image045-1544403281341.jpg)

（3）注意：如果eclipse打印不出日志，在控制台上只显示

   1.log4j:WARN No appenders could be found for logger (org.apache.hadoop.util.Shell).     2.log4j:WARN Please initialize the log4j system properly.     3.log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.   

需要在项目的src目录下，新建一个文件，命名为“log4j.properties”，在文件中填入

   log4j.rootLogger=INFO, stdout     log4j.appender.stdout=org.apache.log4j.ConsoleAppender     log4j.appender.stdout.layout=org.apache.log4j.PatternLayout     log4j.appender.stdout.layout.ConversionPattern=%d   %p [%c] - %m%n     log4j.appender.logfile=org.apache.log4j.FileAppender     log4j.appender.logfile.File=target/spring.log     log4j.appender.logfile.layout=org.apache.log4j.PatternLayout     log4j.appender.logfile.layout.ConversionPattern=%d   %p [%c] - %m%n     

## 3.2 流量汇总程序案例

### 3.2.1 需求1：统计手机号耗费的总上行流量、下行流量、总流量（序列化）

1）需求：

统计每一个手机号耗费的总上行流量、下行流量、总流量

2）数据准备

![img](D:\software\Typora\imgs\clip_image047-1544403281342.png)

输入数据格式：

   1363157993055              13560436666   C4-17-FE-BA-DE-D9:CMCC  120.196.100.99              18         15         1116                   954                     200                               手机号码                                                                                         上行流量 下行流量   

 

输出数据格式

   13560436666                1116                         954                                   2070   手机号码               上行流量        下行流量                总流量   

 

3）分析

基本思路：

Map阶段：

（1）读取一行数据，切分字段

（2）抽取手机号、上行流量、下行流量

（3）以手机号为key，bean对象为value输出，即context.write(手机号,bean);

Reduce阶段：

（1）累加上行流量和下行流量得到总流量。

（2）实现自定义的bean来封装流量信息，并将bean作为map输出的key来传输

（3）MR程序在处理数据的过程中会对数据排序(map输出的kv对传输到reduce之前，会排序)，排序的依据是map输出的key

所以，我们如果要实现自己需要的排序规则，则可以考虑将排序因素放到key中，让key实现接口：WritableComparable。

然后重写key的compareTo方法。

4）编写mapreduce程序

​       （1）编写流量统计的bean对象

   **package** com.atguigu.mr.flowsum;   **import** java.io.DataInput;   **import** java.io.DataOutput;   **import** java.io.IOException;   **import** org.apache.hadoop.io.Writable;       // bean对象要实例化   **public** **class** FlowBean **implements** Writable {              **private** **long** upFlow;          **private** **long** downFlow;          **private** **long** sumFlow;              // 反序列化时，需要反射调用空参构造函数，所以必须有          **public** FlowBean() {                 **super**();          }              **public** FlowBean(**long**   upFlow, **long** downFlow) {                 **super**();                 **this**.upFlow = upFlow;                 **this**.downFlow = downFlow;                 **this**.sumFlow = upFlow +   downFlow;          }              **public** **long** getSumFlow()   {                 **return** sumFlow;          }              **public** **void** setSumFlow(**long**   sumFlow) {                 **this**.sumFlow = sumFlow;          }              **public** **long** getUpFlow() {                 **return** upFlow;          }              **public** **void** setUpFlow(**long**   upFlow) {                 **this**.upFlow = upFlow;          }              **public** **long** getDownFlow()   {                 **return** downFlow;          }              **public** **void** setDownFlow(**long**   downFlow) {                 **this**.downFlow = downFlow;          }              /**           * 序列化方法           *            * **@param** out           * **@throws** IOException           */          @Override          **public** **void**   write(DataOutput out) **throws** IOException {                 out.writeLong(upFlow);                 out.writeLong(downFlow);                 out.writeLong(sumFlow);          }              /**           * 反序列化方法    注意反序列化的顺序和序列化的顺序完全一致           *            * **@param** in           * **@throws** IOException           */          @Override          **public** **void**   readFields(DataInput in) **throws** IOException {                 upFlow = in.readLong();                 downFlow = in.readLong();                 sumFlow = in.readLong();          }              @Override          **public** String toString() {                 **return** upFlow +   "\t" + downFlow + "\t" + sumFlow;          }   }   

​       （2）编写mapreduce主程序

   **package** com.atguigu.mr.flowsum;   **import** java.io.IOException;   **import** org.apache.hadoop.conf.Configuration;   **import** org.apache.hadoop.fs.Path;   **import** org.apache.hadoop.io.LongWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Job;   **import** org.apache.hadoop.mapreduce.Mapper;   **import** org.apache.hadoop.mapreduce.Reducer;   **import**   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import**   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       **public** **class** FlowCount {              **static** **class**   FlowCountMapper **extends** Mapper<LongWritable, Text, Text,   FlowBean> {                     @Override                 **protected** **void**   map(LongWritable key, Text value, Context context) **throws** IOException,   InterruptedException {                        // 1 将一行内容转成string                        String ling =   value.toString();                            // 2 切分字段                        String[] fields =   ling.split("\t");                            // 3 取出手机号码                        String phoneNum =   fields[1];                            // 4 取出上行流量和下行流量                        **long** upFlow =   Long.*parseLong*(fields[fields.length - 3]);                        **long** downFlow = Long.*parseLong*(fields[fields.length   - 2]);                            // 5 写出数据                        context.write(**new**   Text(phoneNum), **new** FlowBean(upFlow, downFlow));                 }          }              **static** **class**   FlowCountReducer **extends** Reducer<Text, FlowBean, Text, FlowBean>   {                 @Override                 **protected** **void**   reduce(Text key, Iterable<FlowBean> values, Context context)                               **throws**   IOException, InterruptedException {                        **long** sum_upFlow =   0;                        **long** sum_downFlow   = 0;                            // 1 遍历所用bean，将其中的上行流量，下行流量分别累加                        **for** (FlowBean bean   : values) {                               sum_upFlow +=   bean.getUpFlow();                               sum_downFlow +=   bean.getDownFlow();                        }                            // 2 封装对象                        FlowBean resultBean = **new**   FlowBean(sum_upFlow, sum_downFlow);                        context.write(key,   resultBean);                 }          }              **public** **static** **void**   main(String[] args) **throws** Exception {                 // 1 获取配置信息，或者job对象实例                 Configuration configuration = **new**   Configuration();                 Job job = Job.*getInstance*(configuration);                     // 6 指定本程序的jar包所在的本地路径                 job.setJarByClass(FlowCount.**class**);                     // 2 指定本业务job要使用的mapper/Reducer业务类                 job.setMapperClass(FlowCountMapper.**class**);                 job.setReducerClass(FlowCountReducer.**class**);                     // 3 指定mapper输出数据的kv类型                 job.setMapOutputKeyClass(Text.**class**);                 job.setMapOutputValueClass(FlowBean.**class**);                     // 4 指定最终输出的数据的kv类型                 job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(FlowBean.**class**);                     // 5 指定job的输入原始文件所在目录                 FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                     // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行                 **boolean** result =   job.waitForCompletion(**true**);                 System.*exit*(result ? 0 : 1);          }   }   

（3）将程序打成jar包，然后拷贝到hadoop集群中。

（4）启动hadoop集群

（5）执行flowcount程序

[atguigu@hadoop102 software]$ hadoop jar flowcount.jar com.atguigu.mr.flowsum.FlowCount /user/atguigu/flowcount/input/ /user/atguigu/flowcount/output

（6）查看结果

[atguigu@hadoop102 software]$ hadoop fs -cat /user/atguigu/flowcount/output/part-r-00000

13480253104  FlowBean [upFlow=180, downFlow=180, sumFlow=360]

13502468823  FlowBean [upFlow=7335, downFlow=110349, sumFlow=117684]

13560436666  FlowBean [upFlow=1116, downFlow=954, sumFlow=2070]

13560439658  FlowBean [upFlow=2034, downFlow=5892, sumFlow=7926]

13602846565  FlowBean [upFlow=1938, downFlow=2910, sumFlow=4848]

。。。

### 3.2.2 需求2：将统计结果按照手机归属地不同省份输出到不同文件中（Partitioner）

0）需求：将统计结果按照手机归属地不同省份输出到不同文件中（分区）

1）数据准备

![img](D:\software\Typora\imgs\clip_image049-1544403281341.png)

2）分析

（1）Mapreduce中会将map输出的kv对，按照相同key分组，然后分发给不同的reducetask。默认的分发规则为：根据key的hashcode%reducetask数来分发

（2）如果要按照我们自己的需求进行分组，则需要改写数据分发（分组）组件Partitioner

自定义一个CustomPartitioner继承抽象类：Partitioner

（3）在job驱动中，设置自定义partitioner： job.setPartitionerClass(CustomPartitioner.class)

3）在需求1的基础上，增加一个分区类

   **package** com.atguigu.mr.partitioner;   **import** java.util.HashMap;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Partitioner;       /**    * K2 V2 对应的是map输出kv类型   * **@author**   Administrator   */   **public** **class** ProvincePartitioner **extends**   Partitioner<Text, FlowBean> {          @Override          **public** **int**   getPartition(Text key, FlowBean value, **int** numPartitions) {   // 1 获取电话号码的前三位                 String preNum =   key.toString().substring(0, 3);                                  **int** partition = 4;                                  // 2 判断是哪个省                 **if**   ("136".equals(preNum)) {                        partition = 0;                 }**else** **if**   ("137".equals(preNum)) {                        partition = 1;                 }**else** **if**   ("138".equals(preNum)) {                        partition = 2;                 }**else** **if** ("139".equals(preNum))   {                        partition = 3;                 }                 **return** partition;          }   }   

2）在驱动函数中增加自定义数据分区设置和reduce task设置

   **public** **static** **void** main(String[] args)   **throws** Exception {                 // 1 获取配置信息，或者job对象实例                 Configuration configuration = **new**   Configuration();                 Job job = Job.*getInstance*(configuration);                     // 6 指定本程序的jar包所在的本地路径                 job.setJarByClass(FlowCount.**class**);                     // 8 指定自定义数据分区                 job.setPartitionerClass(ProvincePartitioner.**class**);                                  // 9 同时指定相应数量的reduce task                 job.setNumReduceTasks(5);                                   // 2 指定本业务job要使用的mapper/Reducer业务类                 job.setMapperClass(FlowCountMapper.**class**);                 job.setReducerClass(FlowCountReducer.**class**);                     // 3 指定mapper输出数据的kv类型                 job.setMapOutputKeyClass(Text.**class**);                 job.setMapOutputValueClass(FlowBean.**class**);                     // 4 指定最终输出的数据的kv类型                 job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(FlowBean.**class**);                     // 5 指定job的输入原始文件所在目录                 FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                     // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行                 **boolean** result =   job.waitForCompletion(**true**);                 System.*exit*(result ? 0 : 1);          }   

3）将程序打成jar包，然后拷贝到hadoop集群中。

4）启动hadoop集群

5）执行flowcountPartitionser程序

[atguigu@hadoop102 software]$ hadoop jar flowcountPartitionser.jar com.atguigu.mr.partitioner.FlowCount /user/atguigu/flowcount/input /user/atguigu/flowcount/output

6）查看结果

[atguigu@hadoop102 software]$ hadoop fs -lsr /

/user/atguigu/flowcount/output/part-r-00000

/user/atguigu/flowcount/output/part-r-00001

/user/atguigu/flowcount/output/part-r-00002

/user/atguigu/flowcount/output/part-r-00003

/user/atguigu/flowcount/output/part-r-00004

### 3.2.3 需求3：将统计结果按照总流量倒序排序（排序）

0）需求

根据需求1产生的结果再次对总流量进行排序。

1）数据准备

![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image049.png)

2）分析

​       （1）把程序分两步走，第一步正常统计总流量，第二步再把结果进行排序

​       （2）context.write(总流量，手机号)

​       （3）FlowBean实现WritableComparable接口重写compareTo方法

   @Override   **public** **int** compareTo(FlowBean o) {          // 倒序排列，从大到小          **return** **this**.sumFlow > o.getSumFlow() ? -1 :   1;   }   

3）FlowBean对象在在需求1基础上增加了比较功能

   **package** com.atguigu.mr.sort;   **import** java.io.DataInput;   **import** java.io.DataOutput;   **import** java.io.IOException;   **import** org.apache.hadoop.io.WritableComparable;       **public** **class** FlowBean **implements** WritableComparable<FlowBean> {              **private** **long** upFlow;          **private** **long** downFlow;          **private** **long** sumFlow;              // 反序列化时，需要反射调用空参构造函数，所以必须有          **public** FlowBean() {                 **super**();          }              **public** FlowBean(**long**   upFlow, **long** downFlow) {                 **super**();                 **this**.upFlow = upFlow;                 **this**.downFlow = downFlow;                 **this**.sumFlow = upFlow +   downFlow;          }              **public** **void** set(**long** upFlow, **long**   downFlow) {                 **this**.upFlow = upFlow;                 **this**.downFlow = downFlow;                 **this**.sumFlow = upFlow + downFlow;          }              **public** **long** getSumFlow()   {                 **return** sumFlow;          }              **public** **void** setSumFlow(**long**   sumFlow) {                 **this**.sumFlow = sumFlow;          }              **public** **long** getUpFlow() {                 **return** upFlow;          }              **public** **void** setUpFlow(**long**   upFlow) {                 **this**.upFlow = upFlow;          }              **public** **long** getDownFlow()   {                 **return** downFlow;          }              **public** **void** setDownFlow(**long**   downFlow) {                 **this**.downFlow = downFlow;          }              /**           * 序列化方法           * **@param** out           * **@throws** IOException           */          @Override          **public** **void**   write(DataOutput out) **throws** IOException {                 out.writeLong(upFlow);                 out.writeLong(downFlow);                 out.writeLong(sumFlow);          }              /**           * 反序列化方法 注意反序列化的顺序和序列化的顺序完全一致           * **@param** in           * **@throws** IOException           */          @Override          **public** **void**   readFields(DataInput in) **throws** IOException {                 upFlow = in.readLong();                 downFlow = in.readLong();                 sumFlow = in.readLong();          }              @Override          **public** String toString() {                 **return** upFlow +   "\t" + downFlow + "\t" + sumFlow;          }              @Override          **public** **int** compareTo(FlowBean o) {                 // 倒序排列，从大到小                 **return** **this**.sumFlow > o.getSumFlow()   ? -1 : 1;          }   }   

4）Map方法优化为一个对象，reduce方法则直接输出结果即可，驱动函数根据输入输出重写配置即可。

   **package** com.atguigu.mr.sort;   **import** java.io.IOException;   **import** org.apache.hadoop.conf.Configuration;   **import** org.apache.hadoop.fs.Path;   **import** org.apache.hadoop.io.LongWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Job;   **import** org.apache.hadoop.mapreduce.Mapper;   **import** org.apache.hadoop.mapreduce.Reducer;   **import**   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import** org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       **public** **class** FlowCountSort {          **static** **class**   FlowCountSortMapper **extends** Mapper<LongWritable, Text, FlowBean,   Text>{                 FlowBean bean = **new**   FlowBean();                 Text v = **new** Text();                                  @Override                 **protected** **void**   map(LongWritable key, Text value, Context context)                               **throws**   IOException, InterruptedException {                                                // 1 拿到的是上一个统计程序输出的结果，已经是各手机号的总流量信息                        String line =   value.toString();                                                // 2 截取字符串并获取电话号、上行流量、下行流量                        String[] fields =   line.split("\t");                        String phoneNbr = fields[0];                                                **long** upFlow =   Long.*parseLong*(fields[1]);                        **long** downFlow =   Long.*parseLong*(fields[2]);                                                // 3 封装对象                        bean.set(upFlow,   downFlow);                        v.set(phoneNbr);                                                // 4 输出                        context.write(bean, v);                 }          }                    **static** **class**   FlowCountSortReducer **extends** Reducer<FlowBean, Text, Text,   FlowBean>{                                  @Override                 **protected** **void**   reduce(FlowBean bean, Iterable<Text> values, Context context)                               **throws**   IOException, InterruptedException {                        context.write(values.iterator().next(),   bean);                 }          }                    **public** **static** **void**   main(String[] args) **throws** Exception {                 // 1 获取配置信息，或者job对象实例                 Configuration configuration = **new**   Configuration();                 Job job = Job.*getInstance*(configuration);                     // 6 指定本程序的jar包所在的本地路径                 job.setJarByClass(FlowCountSort.**class**);                     // 2 指定本业务job要使用的mapper/Reducer业务类                 job.setMapperClass(FlowCountSortMapper.**class**);                 job.setReducerClass(FlowCountSortReducer.**class**);                     // 3 指定mapper输出数据的kv类型                 job.setMapOutputKeyClass(FlowBean.**class**);                 job.setMapOutputValueClass(Text.**class**);                     // 4 指定最终输出的数据的kv类型                 job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(FlowBean.**class**);                     // 5 指定job的输入原始文件所在目录                 FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                                  Path outPath = **new**   Path(args[1]);   //            FileSystem fs =   FileSystem.get(configuration);   //            if (fs.exists(outPath)) {   //                   fs.delete(outPath, true);   //            }                 FileOutputFormat.*setOutputPath*(job,   outPath);                     // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行                 **boolean** result =   job.waitForCompletion(**true**);                 System.*exit*(result ? 0 : 1);          }   }   

5）将程序打成jar包，然后拷贝到hadoop集群中。

6）启动hadoop集群

7）执行flowcountsort程序

[atguigu@hadoop102 software]$ hadoop jar flowcountsort.jar com.atguigu.mr.sort.FlowCountSort /user/atguigu/flowcount/output /user/atguigu/flowcount/output_sort

8）查看结果

[atguigu@hadoop102 software]$ hadoop fs -cat /user/atguigu/flowcount/output_sort/part-r-00000

13502468823  7335       110349    117684

13925057413  11058      48243     59301

13726238888  2481       24681     27162

13726230503  2481       24681     27162

18320173382  9531       2412       11943

## 3.3 求每个订单中最贵的商品（GroupingComparator）

1）需求

有如下订单数据

| 订单id        | 商品id | 成交金额 |
| ------------- | ------ | -------- |
| Order_0000001 | Pdt_01 | 222.8    |
| Order_0000001 | Pdt_05 | 25.8     |
| Order_0000002 | Pdt_03 | 522.8    |
| Order_0000002 | Pdt_04 | 122.4    |
| Order_0000002 | Pdt_05 | 722.4    |
| Order_0000003 | Pdt_01 | 222.8    |
| Order_0000003 | Pdt_02 | 33.8     |

现在需要求出每一个订单中最贵的商品。

2）输入数据

![img](D:\software\Typora\imgs\clip_image052-1544403281342.png)

输出数据预期：

![img](D:\software\Typora\imgs\clip_image054-1544403281342.png)  ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image056.png)  ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image058.png)

3）分析

（1）利用“订单id和成交金额”作为key，可以将map阶段读取到的所有订单数据按照id分区，按照金额排序，发送到reduce。

（2）在reduce端利用groupingcomparator将订单id相同的kv聚合成组，然后取第一个即是最大值。

![img](D:\software\Typora\imgs\clip_image060-1544403281342.png)

4）实现

定义订单信息OrderBean

   **package**   com.atguigu.mapreduce.order;   **import**   java.io.DataInput;   **import**   java.io.DataOutput;   **import**   java.io.IOException;   **import**   org.apache.hadoop.io.WritableComparable;       **public** **class**   OrderBean **implements** WritableComparable<OrderBean> {              **private**   String orderId;          **private**   **double** price;              **public**   OrderBean() {                 **super**();          }              **public**   OrderBean(String orderId, **double** price) {                 **super**();                 **this**.orderId   = orderId;                 **this**.price   = price;          }              **public**   String getOrderId() {                 **return**   orderId;          }              **public**   **void** setOrderId(String orderId) {                 **this**.orderId   = orderId;          }              **public**   **double** getPrice() {                 **return**   price;          }              **public**   **void** setPrice(**double** price) {                 **this**.price   = price;          }              @Override          **public**   **void** readFields(DataInput in) **throws** IOException {                 **this**.orderId   = in.readUTF();                 **this**.price   = in.readDouble();          }              @Override          **public**   **void** write(DataOutput out) **throws** IOException {                 out.writeUTF(orderId);                 out.writeDouble(price);          }              @Override          **public**   **int** compareTo(OrderBean o) {                 //   1 先按订单id排序(从小到大)                 **int**   result = **this**.orderId.compareTo(o.getOrderId());                     **if**   (result == 0) {                        //   2 再按金额排序（从大到小）                        result   = price > o.getPrice() ? -1 : 1;                 }                     **return**   result;          }          @Override          **public**   String toString() {                 **return**   orderId + "\t" + price ;          }   }   

编写OrderSortMapper处理流程

   **package**   com.atguigu.mapreduce.order;   **import**   java.io.IOException;   **import**   org.apache.hadoop.io.LongWritable;   **import**   org.apache.hadoop.io.NullWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Mapper;       **public** **class**   OrderSortMapper **extends** Mapper<LongWritable, Text, OrderBean,   NullWritable>{          OrderBean   bean = **new** OrderBean();                    @Override          **protected**   **void** map(LongWritable key, Text value,                        Context   context)**throws** IOException, InterruptedException {                 //   1 获取一行数据                 String   line = value.toString();                                  //   2 截取字段                 String[]   fields = line.split("\t");                                  //   3 封装bean                 bean.setOrderId(fields[0]);                 bean.setPrice(Double.*parseDouble*(fields[2]));                                  //   4 写出                 context.write(bean,   NullWritable.*get*());          }   }   

编写OrderSortReducer处理流程

   package com.atguigu.mapreduce.order;   import java.io.IOException;   import org.apache.hadoop.io.NullWritable;   import   org.apache.hadoop.mapreduce.Reducer;       public class OrderSortReducer extends   Reducer<OrderBean, NullWritable, OrderBean, NullWritable>{          @Override          protected   void reduce(OrderBean bean, Iterable<NullWritable> values,                        Context   context) throws IOException, InterruptedException {                 //   直接写出                 context.write(bean,   NullWritable.get());          }   }   

编写OrderSortDriver处理流程

   package com.atguigu.mapreduce.order;   import   org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class OrderSortDriver {              public   static void main(String[] args) throws Exception {                 //   1 获取配置信息                 Configuration   conf = new Configuration();                 Job   job = Job.getInstance(conf);                     //   2 设置jar包加载路径                 job.setJarByClass(OrderSortDriver.class);                     //   3 加载map/reduce类                 job.setMapperClass(OrderSortMapper.class);                 job.setReducerClass(OrderSortReducer.class);                     //   4 设置map输出数据key和value类型                 job.setMapOutputKeyClass(OrderBean.class);                 job.setMapOutputValueClass(NullWritable.class);                     //   5 设置最终输出数据的key和value类型                 job.setOutputKeyClass(OrderBean.class);                 job.setOutputValueClass(NullWritable.class);                     //   6 设置输入数据和输出数据路径                 FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     //   10 设置reduce端的分组                 job.setGroupingComparatorClass(OrderSortGroupingComparator.class);                                  //   7 设置分区                 job.setPartitionerClass(OrderSortPartitioner.class);                                  //   8 设置reduce个数                 job.setNumReduceTasks(3);                     //   9 提交                 boolean   result = job.waitForCompletion(true);                 System.exit(result   ? 0 : 1);          }   }   

编写OrderSortPartitioner处理流程

   package com.atguigu.mapreduce.order;   import org.apache.hadoop.io.NullWritable;   import   org.apache.hadoop.mapreduce.Partitioner;       public class OrderSortPartitioner extends   Partitioner<OrderBean, NullWritable>{              @Override          public   int getPartition(OrderBean key, NullWritable value, int numReduceTasks) {                                  return   (key.getOrderId().hashCode() & Integer.MAX_VALUE) % numReduceTasks;          }   }   

编写OrderSortGroupingComparator处理流程

   package com.atguigu.mapreduce.order;   import   org.apache.hadoop.io.WritableComparable;   import   org.apache.hadoop.io.WritableComparator;       public class OrderSortGroupingComparator   extends WritableComparator {              protected OrderSortGroupingComparator() {                 super(OrderBean.class, true);          }              @Override          public   int compare(WritableComparable a, WritableComparable b) {                                  OrderBean   abean = (OrderBean) a;                 OrderBean   bbean = (OrderBean) b;                                  //   将orderId相同的bean都视为一组                 return   abean.getOrderId().compareTo(bbean.getOrderId());          }   }   

## 3.4 MapReduce中多表合并案例

1）需求：

订单数据表t_order：

| id   | pid  | amount |
| ---- | ---- | ------ |
| 1001 | 01   | 1      |
| 1002 | 02   | 2      |
| 1003 | 03   | 3      |

![img](D:\software\Typora\imgs\clip_image062-1544403281342.png)

商品信息表t_product

| id   | pname |
| ---- | ----- |
| 01   | 小米  |
| 02   | 华为  |
| 03   | 格力  |

![img](D:\software\Typora\imgs\clip_image064-1544403281342.png)

​       将商品信息表中数据根据商品id合并到订单数据表中。

最终数据形式：

| id   | pname | amount |
| ---- | ----- | ------ |
| 1001 | 小米  | 1      |
| 1001 | 小米  | 1      |
| 1002 | 华为  | 2      |
| 1002 | 华为  | 2      |
| 1003 | 格力  | 3      |
| 1003 | 格力  | 3      |

### 3.4.1 需求1：reduce端表合并（数据倾斜）

通过将关联条件作为map输出的key，将两表满足join条件的数据并携带数据所来源的文件信息，发往同一个reduce task，在reduce中进行数据的串联。

![img](D:\software\Typora\imgs\clip_image066-1544403281342.png)

1）创建商品和订合并后的bean类

   package com.atguigu.mapreduce.table;   import java.io.DataInput;   import java.io.DataOutput;   import java.io.IOException;   import org.apache.hadoop.io.Writable;       public class TableBean implements Writable {          private String   order_id; // 订单id          private String p_id;   // 产品id          private int amount;   // 产品数量          private String   pname; // 产品名称          private String   flag;// 表的标记              public TableBean() {                 super();          }              public   TableBean(String order_id, String p_id, int amount, String pname, String   flag) {                 super();                 this.order_id   = order_id;                 this.p_id =   p_id;                 this.amount =   amount;                 this.pname =   pname;                 this.flag =   flag;          }              public String   getFlag() {                 return flag;          }              public void   setFlag(String flag) {                 this.flag =   flag;          }              public String   getOrder_id() {                 return   order_id;          }              public void   setOrder_id(String order_id) {                 this.order_id   = order_id;          }              public String   getP_id() {                 return p_id;          }              public void   setP_id(String p_id) {                 this.p_id =   p_id;          }              public int   getAmount() {                 return   amount;          }              public void   setAmount(int amount) {                 this.amount =   amount;          }              public String   getPname() {                 return pname;          }              public void   setPname(String pname) {                 this.pname =   pname;          }              @Override          public void   write(DataOutput out) throws IOException {                 out.writeUTF(order_id);                 out.writeUTF(p_id);                 out.writeInt(amount);                 out.writeUTF(pname);                 out.writeUTF(flag);          }              @Override          public void   readFields(DataInput in) throws IOException {                 this.order_id   = in.readUTF();                 this.p_id =   in.readUTF();                 this.amount =   in.readInt();                 this.pname =   in.readUTF();                 this.flag =   in.readUTF();          }              @Override          public String   toString() {                 return   order_id + "\t" + p_id + "\t" + amount + "\t" ;          }   }   

2）编写TableMapper程序

   **package** com.atguigu.mapreduce.table;   **import** java.io.IOException;   **import** org.apache.hadoop.io.LongWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Mapper;   **import** org.apache.hadoop.mapreduce.lib.input.FileSplit;       **public** **class** TableMapper **extends**   Mapper<LongWritable, Text, Text, TableBean>{          TableBean bean = **new**   TableBean();          Text k = **new**   Text();                    @Override          **protected** **void**   map(LongWritable key, Text value, Context context)                        **throws**   IOException, InterruptedException {                                  // 1 获取输入文件类型                 FileSplit   split = (FileSplit) context.getInputSplit();                 String name =   split.getPath().getName();                                  // 2 获取输入数据                 String line =   value.toString();                                  // 3 不同文件分别处理                 **if**   (name.startsWith("order")) {// 订单表处理                        // 3.1   切割                        String[]   fields = line.split(",");                                                // 3.2   封装bean对象                        bean.setOrder_id(fields[0]);                        bean.setP_id(fields[1]);                        bean.setAmount(Integer.*parseInt*(fields[2]));                        bean.setPname("");                        bean.setFlag("0");                                                k.set(fields[1]);                 }**else**   {// 产品表处理                        // 3.3   切割                        String[]   fields = line.split(",");                                                // 3.4   封装bean对象                        bean.setP_id(fields[0]);                        bean.setPname(fields[1]);                        bean.setFlag("1");                        bean.setAmount(0);                        bean.setOrder_id("");                                                k.set(fields[0]);                 }                 // 4 写出                 context.write(k,   bean);          }   }   

3）编写TableReducer程序

   package com.atguigu.mapreduce.table;   import java.io.IOException;   import java.util.ArrayList;   import org.apache.commons.beanutils.BeanUtils;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Reducer;       public class TableReducer extends Reducer<Text, TableBean,   TableBean, NullWritable> {              @Override          protected void   reduce(Text key, Iterable<TableBean> values, Context context)                        throws   IOException, InterruptedException {                     // 1准备存储订单的集合                 ArrayList<TableBean>   orderBeans = new ArrayList<>();                 // 2 准备bean对象                 TableBean   pdBean = new TableBean();                     for   (TableBean bean : values) {                            if   ("0".equals(bean.getFlag())) {// 订单表                               //   拷贝传递过来的每条订单数据到集合中                               TableBean   orderBean = new TableBean();                               try   {                                      BeanUtils.copyProperties(orderBean,   bean);                               }   catch (Exception e) {                                      e.printStackTrace();                               }                                   orderBeans.add(orderBean);                        } else   {// 产品表                               try   {                                      //   拷贝传递过来的产品表到内存中                                      BeanUtils.copyProperties(pdBean,   bean);                               }   catch (Exception e) {                                      e.printStackTrace();                               }                        }                 }                     // 3 表的拼接                 for(TableBean   bean:orderBeans){                        bean.setP_id(pdBean.getPname());                                                // 4 数据写出去                        context.write(bean,   NullWritable.get());                 }          }   }   

4）编写TableDriver程序

   package com.atguigu.mapreduce.table;       import org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Job;   import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class TableDriver {              public static void   main(String[] args) throws Exception {                 // 1 获取配置信息，或者job对象实例                 Configuration   configuration = new Configuration();                 Job job =   Job.getInstance(configuration);                     // 2 指定本程序的jar包所在的本地路径                 job.setJarByClass(TableDriver.class);                     // 3 指定本业务job要使用的mapper/Reducer业务类                 job.setMapperClass(TableMapper.class);                 job.setReducerClass(TableReducer.class);                     // 4 指定mapper输出数据的kv类型                 job.setMapOutputKeyClass(Text.class);                 job.setMapOutputValueClass(TableBean.class);                     // 5 指定最终输出的数据的kv类型                 job.setOutputKeyClass(TableBean.class);                 job.setOutputValueClass(NullWritable.class);                     // 6 指定job的输入原始文件所在目录                 FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行                 boolean   result = job.waitForCompletion(true);                 System.exit(result   ? 0 : 1);          }   }   

3）运行程序查看结果

   1001       小米       1        1001       小米       1        1002       华为       2        1002       华为       2        1003       格力       3        1003       格力       3        

缺点：这种方式中，合并的操作是在reduce阶段完成，reduce端的处理压力太大，map节点的运算负载则很低，资源利用率不高，且在reduce阶段极易产生数据倾斜

解决方案： map端实现数据合并

### 3.4.2 需求2：map端表合并（Distributedcache）

1）分析

适用于关联表中有小表的情形；

可以将小表分发到所有的map节点，这样，map节点就可以在本地对自己所读到的大表数据进行合并并输出最终结果，可以大大提高合并操作的并发度，加快处理速度。

![img](D:\software\Typora\imgs\clip_image068-1544403281342.png)

2）实操案例

（1）先在驱动模块中添加缓存文件

   package test;   import java.net.URI;   import   org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import   org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class DistributedCacheDriver {              public   static void main(String[] args) throws Exception {                 //   1 获取job信息                 Configuration   configuration = new Configuration();                 Job   job = Job.getInstance(configuration);                     //   2 设置加载jar包路径                 job.setJarByClass(DistributedCacheDriver.class);                     //   3 关联map                 job.setMapperClass(DistributedCacheMapper.class);                                  //   4 设置最终输出数据类型                 job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(NullWritable.class);                     //   5 设置输入输出路径                 FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     //   6 加载缓存数据                 job.addCacheFile(new   URI("file:/e:/cache/pd.txt"));                                  //   7 map端join的逻辑不需要reduce阶段，设置reducetask数量为0                 job.setNumReduceTasks(0);                     //   8 提交                 boolean   result = job.waitForCompletion(true);                 System.exit(result   ? 0 : 1);          }   }   

（2）读取缓存的文件数据

   package test;   import java.io.BufferedReader;   import java.io.FileInputStream;   import java.io.IOException;   import java.io.InputStreamReader;   import java.util.HashMap;   import java.util.Map;   import   org.apache.commons.lang.StringUtils;   import   org.apache.hadoop.io.LongWritable;   import   org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Mapper;       public class DistributedCacheMapper   extends Mapper<LongWritable, Text, Text, NullWritable>{              Map<String,   String> pdMap = new HashMap<>();                    @Override          protected   void setup(Mapper<LongWritable, Text, Text, NullWritable>.Context context)                        throws   IOException, InterruptedException {                 //   1 获取缓存的文件                 BufferedReader   reader = new BufferedReader(new InputStreamReader(new   FileInputStream("pd.txt")));                                  String   line;                 while(StringUtils.isNotEmpty(line   = reader.readLine())){                        //   2 切割                        String[]   fields = line.split("\t");                                                //   3 缓存数据到集合                        pdMap.put(fields[0],   fields[1]);                 }                                  //   4 关流                 reader.close();          }                    Text   k = new Text();                    @Override          protected   void map(LongWritable key, Text value, Context context)                        throws   IOException, InterruptedException {                 //   1 获取一行                 String   line = value.toString();                                  //   2 截取                 String[]   fields = line.split("\t");                                  //   3 获取订单id                 String   orderId = fields[1];                                  //   4 获取商品名称                 String   pdName = pdMap.get(orderId);                                  //   5 拼接                 k.set(line   + "\t"+ pdName);                                  //   6 写出                 context.write(k,   NullWritable.get());          }   }   

## 3.5 小文件处理（自定义InputFormat）

1）需求

无论hdfs还是mapreduce，对于小文件都有损效率，实践中，又难免面临处理大量小文件的场景，此时，就需要有相应解决方案。将多个小文件合并成一个文件SequenceFile，SequenceFile里面存储着多个文件，存储的形式为文件路径+名称为key，文件内容为value。

2）输入数据

![img](D:\software\Typora\imgs\clip_image070-1544403281342.png)  ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image072.png)   ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image074.png)

最终预期文件格式：

![img](D:\software\Typora\imgs\clip_image076-1544403281342.png)

3）分析

小文件的优化无非以下几种方式：

（1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS

（2）在业务处理之前，在HDFS上使用mapreduce程序对小文件进行合并

（3）在mapreduce处理时，可采用CombineTextInputFormat提高效率

4）具体实现

本节采用自定义InputFormat的方式，处理输入小文件的问题。

（1）自定义一个InputFormat

（2）改写RecordReader，实现一次读取一个完整文件封装为KV

（3）在输出时使用SequenceFileOutPutFormat输出合并文件

5）程序实现：

（1）自定义InputFromat

   package   com.atguigu.mapreduce.inputformat;   import java.io.IOException;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.BytesWritable;   import org.apache.hadoop.io.NullWritable;   import   org.apache.hadoop.mapreduce.InputSplit;   import   org.apache.hadoop.mapreduce.JobContext;   import   org.apache.hadoop.mapreduce.RecordReader;   import org.apache.hadoop.mapreduce.TaskAttemptContext;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;       public class WholeFileInputformat extends   FileInputFormat<NullWritable, BytesWritable>{              @Override          protected   boolean isSplitable(JobContext context, Path filename) {                 return   false;          }                    @Override          public   RecordReader<NullWritable, BytesWritable> createRecordReader(InputSplit   split, TaskAttemptContext context)                        throws   IOException, InterruptedException {                 //   1 定义一个自己的recordReader                 WholeRecordReader   recordReader = new WholeRecordReader();                                  //   2 初始化recordReader                 recordReader.initialize(split,   context);                                  return   recordReader;          }   }   

（2）自定义RecordReader

   package   com.atguigu.mapreduce.inputformat;   import   java.io.IOException;   import   org.apache.hadoop.conf.Configuration;   import   org.apache.hadoop.fs.FSDataInputStream;   import   org.apache.hadoop.fs.FileSystem;   import   org.apache.hadoop.fs.Path;   import   org.apache.hadoop.io.BytesWritable;   import   org.apache.hadoop.io.IOUtils;   import   org.apache.hadoop.io.NullWritable;   import   org.apache.hadoop.mapreduce.InputSplit;   import   org.apache.hadoop.mapreduce.RecordReader;   import   org.apache.hadoop.mapreduce.TaskAttemptContext;   import   org.apache.hadoop.mapreduce.lib.input.FileSplit;       public   class WholeRecordReader extends RecordReader<NullWritable,   BytesWritable> {          private FileSplit split;          private Configuration configuration;              private BytesWritable value = new   BytesWritable();          private boolean processed = false;              @Override          public void initialize(InputSplit   split, TaskAttemptContext context) throws IOException, InterruptedException {                 // 获取传递过来的数据                 this.split = (FileSplit) split;                 configuration =   context.getConfiguration();          }              @Override          public boolean nextKeyValue() throws   IOException, InterruptedException {                                  if (!processed) {                        // 1 定义缓存                        byte[] contents = new   byte[(int) split.getLength()];                            // 2 获取文件系统                        Path path =   split.getPath();                        FileSystem fs =   path.getFileSystem(configuration);                            // 3 读取内容                        FSDataInputStream fis =   null;                        try {                               // 3.1 打开输入流                               fis = fs.open(path);                                                              // 3.2 读取文件内容                               IOUtils.readFully(fis,   contents, 0, contents.length);                                                              // 3.3 输出文件内容                               value.set(contents,   0, contents.length);                        } catch (Exception e) {                            } finally {                               IOUtils.closeStream(fis);                        }                                                processed = true;                                                return true;                 }                                  return false;          }              @Override          public NullWritable getCurrentKey()   throws IOException, InterruptedException {                                  return NullWritable.get();          }              @Override          public BytesWritable getCurrentValue()   throws IOException, InterruptedException {                                  return value;          }              @Override          public float getProgress() throws   IOException, InterruptedException {                 return processed?1:0;          }              @Override          public void close() throws IOException   {              }   }   

（3）InputFormatDriver处理流程

   package com.atguigu.mapreduce.inputformat;   import java.io.IOException;   import   org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import   org.apache.hadoop.io.BytesWritable;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.InputSplit;   import org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.Mapper;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import   org.apache.hadoop.mapreduce.lib.input.FileSplit;   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;   import   org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;   public class InputFormatDriver {              static   class SequenceFileMapper extends Mapper<NullWritable, BytesWritable, Text,   BytesWritable> {                 private   Text filenameKey;                     @Override                 protected   void setup(Context context) throws IOException, InterruptedException {                        //   获取切片信息                        InputSplit   split = context.getInputSplit();                        //   获取切片路径                        Path   path = ((FileSplit) split).getPath();                        //   根据切片路径获取文件名称                        filenameKey = new   Text(path.toString());                 }                     @Override                 protected   void map(NullWritable key, BytesWritable value, Context context)                               throws   IOException, InterruptedException {                        //   文件名称为key                        context.write(filenameKey,   value);                 }          }              public   static void main(String[] args) throws Exception {                 args   = new String[] { "e:/input", "e:/output11" };                     Configuration   conf = new Configuration();                                  Job   job = Job.getInstance(conf);                 job.setJarByClass(InputFormatDriver.class);                     job.setInputFormatClass(WholeFileInputFormat.class);                 job.setOutputFormatClass(SequenceFileOutputFormat.class);                     job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(BytesWritable.class);                     job.setMapperClass(SequenceFileMapper.class);                     FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     boolean   result = job.waitForCompletion(true);                     System.exit(result   ? 0 : 1);          }   }   

## 3.6 过滤日志及自定义日志输出路径（自定义OutputFormat）

1）需求

​       过滤输入的log日志中是否包含atguigu

​       （1）包含atguigu的网站输出到e:/atguigu.log

​       （2）不包含atguigu的网站输出到e:/other.log

2）输入数据

![img](D:\software\Typora\imgs\clip_image078-1544403281342.png)

输出预期：

![img](D:\software\Typora\imgs\clip_image080-1544403281342.png)  ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image082.png)

3）具体程序：

（1）自定义一个outputformat

   package   com.atguigu.mapreduce.outputformat;   import java.io.IOException;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.RecordWriter;   import   org.apache.hadoop.mapreduce.TaskAttemptContext;   import   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class FilterOutputFormat extends FileOutputFormat<Text,   NullWritable>{              @Override          public   RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext   job)                        throws   IOException, InterruptedException {                     //   创建一个RecordWriter                 return   new FilterRecordWriter(job);          }   }   

（2）具体的写数据RecordWriter

   package   com.atguigu.mapreduce.outputformat;   import java.io.IOException;   import   org.apache.hadoop.fs.FSDataOutputStream;   import org.apache.hadoop.fs.FileSystem;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.RecordWriter;   import   org.apache.hadoop.mapreduce.TaskAttemptContext;       public class FilterRecordWriter extends   RecordWriter<Text, NullWritable> {          FSDataOutputStream   atguiguOut = null;          FSDataOutputStream   otherOut = null;              public   FilterRecordWriter(TaskAttemptContext job) {                 //   1 获取文件系统                 FileSystem   fs;                     try   {                        fs   = FileSystem.get(job.getConfiguration());                            //   2 创建输出文件路径                        Path   atguiguPath = new Path("e:/atguigu.log");                        Path   otherPath = new Path("e:/other.log");                            //   3 创建输出流                        atguiguOut   = fs.create(atguiguPath);                        otherOut   = fs.create(otherPath);                 }   catch (IOException e) {                        e.printStackTrace();                 }          }              @Override          public   void write(Text key, NullWritable value) throws IOException, InterruptedException   {                     //   判断是否包含“atguigu”输出到不同文件                 if   (key.toString().contains("atguigu")) {                        atguiguOut.write(key.toString().getBytes());                 }   else {                        otherOut.write(key.toString().getBytes());                 }          }              @Override          public   void close(TaskAttemptContext context) throws IOException,   InterruptedException {                 //   关闭资源                 if   (atguiguOut != null) {                        atguiguOut.close();                 }                                  if   (otherOut != null) {                        otherOut.close();                 }          }   }   

（3）编写FilterMapper

   package   com.atguigu.mapreduce.outputformat;   import java.io.IOException;   import org.apache.hadoop.io.LongWritable;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.Mapper;       public class FilterMapper extends   Mapper<LongWritable, Text, Text, NullWritable>{                    Text   k = new Text();                    @Override          protected   void map(LongWritable key, Text value, Context context)                        throws   IOException, InterruptedException {                 //   1 获取一行                 String   line = value.toString();                                  k.set(line);                                  //   3 写出                 context.write(k,   NullWritable.get());          }   }   

（4）编写FilterReducer

   package   com.atguigu.mapreduce.outputformat;   import java.io.IOException;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.Reducer;       public class FilterReducer extends   Reducer<Text, NullWritable, Text, NullWritable> {              @Override          protected   void reduce(Text key, Iterable<NullWritable> values, Context context)                        throws   IOException, InterruptedException {                     String   k = key.toString();                 k   = k + "\r\n";                     context.write(new   Text(k), NullWritable.get());          }   }   

（5）编写FilterDriver

   package   com.atguigu.mapreduce.outputformat;   import   org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class FilterDriver {          public   static void main(String[] args) throws Exception {                 Configuration   conf = new Configuration();                     Job   job = Job.getInstance(conf);                     job.setJarByClass(FilterDriver.class);                 job.setMapperClass(FilterMapper.class);                 job.setReducerClass(FilterReducer.class);                     job.setMapOutputKeyClass(Text.class);                 job.setMapOutputValueClass(NullWritable.class);                                  job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(NullWritable.class);                     //   要将自定义的输出格式组件设置到job中                 job.setOutputFormatClass(FilterOutputFormat.class);                     FileInputFormat.setInputPaths(job,   new Path(args[0]));                     //   虽然我们自定义了outputformat，但是因为我们的outputformat继承自fileoutputformat                 //   而fileoutputformat要输出一个_SUCCESS文件，所以，在这还得指定一个输出目录                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     boolean   result = job.waitForCompletion(true);                 System.exit(result   ? 0 : 1);          }   }   

## 3.7 日志清洗（数据清洗）

### 3.7.1 简单解析版

1）需求：

去除日志中字段长度小于等于11的日志。

2）输入数据

![img](D:\software\Typora\imgs\clip_image084-1544403281342.png)

3）实现代码：

（1）编写LogMapper 

   **package**   com.atguigu.mapreduce.weblog;   **import**   java.io.IOException;   **import**   org.apache.hadoop.io.LongWritable;   **import**   org.apache.hadoop.io.NullWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Mapper;       **public** **class**   LogMapper **extends** Mapper<LongWritable, Text, Text,   NullWritable>{                    Text   k = **new** Text();                    @Override          **protected**   **void** map(LongWritable key, Text value, Context context)                        **throws**   IOException, InterruptedException {                                  //   1 获取1行数据                 String   line = value.toString();                                  //   2 解析日志                 **boolean**   result = parseLog(line,context);                                  //   3 日志不合法退出                 **if**   (!result) {                        **return**;                 }                                  //   4 设置key                 k.set(line);                                  //   5 写出数据                 context.write(k,   NullWritable.*get*());          }              //   2 解析日志          **private**   **boolean** parseLog(String line, Context context) {                 //   1 截取                 String[]   fields = line.split(" ");                                  //   2 日志长度大于11的为合法                 **if**   (fields.length > 11) {                        //   系统计数器                        context.getCounter("map",   "true").increment(1);                        **return**   **true**;                 }**else**   {                        context.getCounter("map",   "false").increment(1);                        **return**   **false**;                 }          }   }   

（2）编写LogDriver

   **package**   com.atguigu.mapreduce.weblog;   **import**   org.apache.hadoop.conf.Configuration;   **import**   org.apache.hadoop.fs.Path;   **import**   org.apache.hadoop.io.NullWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Job;   **import**   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import**   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       **public** **class**   LogDriver {              **public**   **static** **void** main(String[] args) **throws** Exception {                 //   1 获取job信息                 Configuration   conf = **new** Configuration();                 Job   job = Job.*getInstance*(conf);                     //   2 加载jar包                 job.setJarByClass(LogDriver.**class**);                     //   3 关联map                 job.setMapperClass(LogMapper.**class**);                     //   4 设置最终输出类型                 job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(NullWritable.**class**);                     //   5 设置输入和输出路径                 FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                     //   6 提交                 job.waitForCompletion(**true**);          }   }   

### 3.7.2 复杂解析版

1）需求：

对web访问日志中的各字段识别切分

去除日志中不合法的记录

根据统计需求，生成各类访问请求过滤数据

2）输入数据

![img](D:\software\Typora\imgs\clip_image086-1544403281342.png)

3）实现代码：

（1）定义一个bean，用来记录日志数据中的各数据字段

   package com.atguigu.mapreduce.log;       public class LogBean {          private   String remote_addr;// 记录客户端的ip地址          private   String remote_user;// 记录客户端用户名称,忽略属性"-"          private   String time_local;// 记录访问时间与时区          private   String request;// 记录请求的url与http协议          private   String status;// 记录请求状态；成功是200          private   String body_bytes_sent;// 记录发送给客户端文件主体内容大小          private   String http_referer;// 用来记录从那个页面链接访问过来的          private   String http_user_agent;// 记录客户浏览器的相关信息              private   boolean valid = true;// 判断数据是否合法              public   String getRemote_addr() {                 return   remote_addr;          }              public   void setRemote_addr(String remote_addr) {                 this.remote_addr   = remote_addr;          }              public   String getRemote_user() {                 return   remote_user;          }              public   void setRemote_user(String remote_user) {                 this.remote_user   = remote_user;          }              public   String getTime_local() {                 return   time_local;          }              public   void setTime_local(String time_local) {                 this.time_local   = time_local;          }              public   String getRequest() {                 return   request;          }              public   void setRequest(String request) {                 this.request   = request;          }              public   String getStatus() {                 return   status;          }              public   void setStatus(String status) {                 this.status   = status;          }              public   String getBody_bytes_sent() {                 return   body_bytes_sent;          }              public   void setBody_bytes_sent(String body_bytes_sent) {                 this.body_bytes_sent   = body_bytes_sent;          }              public   String getHttp_referer() {                 return   http_referer;          }              public   void setHttp_referer(String http_referer) {                 this.http_referer   = http_referer;          }              public   String getHttp_user_agent() {                 return   http_user_agent;          }              public   void setHttp_user_agent(String http_user_agent) {                 this.http_user_agent   = http_user_agent;          }              public   boolean isValid() {                 return   valid;          }              public   void setValid(boolean valid) {                 this.valid   = valid;          }              @Override          public   String toString() {                 StringBuilder   sb = new StringBuilder();                 sb.append(this.valid);                 sb.append("\001").append(this.remote_addr);                 sb.append("\001").append(this.remote_user);                 sb.append("\001").append(this.time_local);                 sb.append("\001").append(this.request);                 sb.append("\001").append(this.status);                 sb.append("\001").append(this.body_bytes_sent);                 sb.append("\001").append(this.http_referer);                 sb.append("\001").append(this.http_user_agent);                                  return   sb.toString();          }   }   

（2）编写LogMapper程序

   package com.atguigu.mapreduce.log;   import java.io.IOException;   import org.apache.hadoop.io.LongWritable;   import   org.apache.hadoop.io.NullWritable;   import org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.Mapper;       public class LogMapper extends   Mapper<LongWritable, Text, Text, NullWritable>{          Text   k = new Text();                    @Override          protected   void map(LongWritable key, Text value, Context context)                        throws   IOException, InterruptedException {                 //   1 获取1行                 String   line = value.toString();                                  //   2 解析日志是否合法                 LogBean   bean = pressLog(line);                                  if   (!bean.isValid()) {                        return;                 }                                  k.set(bean.toString());                                  //   3 输出                 context.write(k,   NullWritable.*get*());          }              //   解析日志          private   LogBean pressLog(String line) {                 LogBean   logBean = new LogBean();                                  //   1 截取                 String[]   fields = line.split(" ");                                  if   (fields.length > 11) {                        //   2封装数据                        logBean.setRemote_addr(fields[0]);                        logBean.setRemote_user(fields[1]);                        logBean.setTime_local(fields[3].substring(1));                        logBean.setRequest(fields[6]);                        logBean.setStatus(fields[8]);                        logBean.setBody_bytes_sent(fields[9]);                        logBean.setHttp_referer(fields[10]);                                                if   (fields.length > 12) {                               logBean.setHttp_user_agent(fields[11]   + " "+ fields[12]);                        }else   {                               logBean.setHttp_user_agent(fields[11]);                        }                                                //   大于400，HTTP错误                        if   (Integer.*parseInt*(logBean.getStatus()) >= 400) {                               logBean.setValid(false);                        }                 }else   {                        logBean.setValid(false);                 }                                  return   logBean;          }   }   

（3）编写LogDriver程序

   **package** com.atguigu.mapreduce.log;   **import** org.apache.hadoop.conf.Configuration;   **import** org.apache.hadoop.fs.Path;   **import** org.apache.hadoop.io.NullWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Job;   **import**   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import**   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       **public** **class** LogDriver {          **public** **static** **void**   main(String[] args) **throws** Exception {                 // 1 获取job信息                 Configuration conf = **new**   Configuration();                 Job job = Job.*getInstance*(conf);                     // 2 加载jar包                 job.setJarByClass(LogDriver.**class**);                     // 3 关联map                 job.setMapperClass(LogMapper.**class**);                     // 4 设置最终输出类型                 job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(NullWritable.**class**);                     // 5 设置输入和输出路径                 FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                     // 6 提交                 job.waitForCompletion(**true**);          }   }   

## 3.8 倒排索引实战（多job串联）

0）需求：有大量的文本（文档、网页），需要建立搜索索引

![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image088.png)  ![img](D:\software\Typora\imgs\clip_image090-1544403281342.png)  ![img](file:///C:/Users/HEYUNX~1/AppData/Local/Temp/msohtmlclip1/01/clip_image092.png)

![img](D:\software\Typora\imgs\clip_image094-1544403281343.jpg)

​       （1）第一次预期输出结果

   atguigu--a.txt  3   atguigu--b.txt  2   atguigu--c.txt  2   pingping--a.txt  1   pingping--b.txt       3   pingping--c.txt  1   ss--a.txt   2   ss--b.txt   1   ss--c.txt   1   

（2）第二次预期输出结果

   atguigu    c.txt-->2  b.txt-->2 a.txt-->3     pingping  c.txt-->1  b.txt-->3 a.txt-->1     ss     c.txt-->1  b.txt-->1 a.txt-->2     

1）第一次处理

（1）第一次处理，编写OneIndexMapper

   **package** com.atguigu.mapreduce.index;   **import** java.io.IOException;   **import** org.apache.hadoop.io.IntWritable;   **import** org.apache.hadoop.io.LongWritable;   **import** org.apache.hadoop.io.Text;   **import** org.apache.hadoop.mapreduce.Mapper;   **import**   org.apache.hadoop.mapreduce.lib.input.FileSplit;       **public** **class** OneIndexMapper **extends**   Mapper<LongWritable, Text, Text, IntWritable> {              Text k = **new** Text();              @Override          **protected** **void**   map(LongWritable key, Text value, Context context) **throws** IOException,   InterruptedException {                 // 1 获取切片名称                 FileSplit inputSplit =   (FileSplit) context.getInputSplit();                 String name =   inputSplit.getPath().getName();                     // 2 获取1行                 String line = value.toString();                     // 3 截取                 String[] words =   line.split(" ");                     // 4 把每个单词和切片名称关联起来                 **for** (String word : words)   {                        k.set(word +   "--" + name);                                                context.write(k, **new**   IntWritable(1));                 }          }   }   

（2）第一次处理，编写OneIndexReducer

   package   com.atguigu.mapreduce.index;   import   java.io.IOException;   import   org.apache.hadoop.io.IntWritable;   import   org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.Reducer;       public class   OneIndexReducer extends Reducer<Text, IntWritable, Text, IntWritable>{                    @Override          protected void reduce(Text key,   Iterable<IntWritable> values,                        Context context) throws   IOException, InterruptedException {                                  int count = 0;                 // 累加和                 for(IntWritable value: values){                        count +=value.get();                 }                                  // 写出                 context.write(key, new   IntWritable(count));          }   }   

（3）第一次处理，编写OneIndexDriver

   package   com.atguigu.mapreduce.index;   import   org.apache.hadoop.conf.Configuration;   import   org.apache.hadoop.fs.Path;   import   org.apache.hadoop.io.IntWritable;   import   org.apache.hadoop.io.Text;   import   org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class   OneIndexDriver {              public static void main(String[] args)   throws Exception {                                  Configuration conf = new   Configuration();                     Job job = Job.getInstance(conf);                 job.setJarByClass(OneIndexDriver.class);                     job.setMapperClass(OneIndexMapper.class);                 job.setReducerClass(OneIndexReducer.class);                     job.setMapOutputKeyClass(Text.class);                 job.setMapOutputValueClass(IntWritable.class);                                  job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(IntWritable.class);                     FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     job.waitForCompletion(true);          }   }   

（4）查看第一次输出结果

   atguigu--a.txt  3   atguigu--b.txt  2   atguigu--c.txt  2   pingping--a.txt 1   pingping--b.txt       3   pingping--c.txt 1   ss--a.txt   2   ss--b.txt   1   ss--c.txt   1   

2）第二次处理

（1）第二次处理，编写TwoIndexMapper

   package com.atguigu.mapreduce.index;   import java.io.IOException;   import org.apache.hadoop.io.LongWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Mapper;       public class TwoIndexMapper extends Mapper<LongWritable,   Text, Text, Text>{          Text k = new   Text();          Text v = new   Text();                    @Override          protected void   map(LongWritable key, Text value, Context context)                        throws   IOException, InterruptedException {                                  // 1 获取1行数据                 String   line = value.toString();                                  // 2用“--”切割                 String[]   fields = line.split("--");                                  k.set(fields[0]);                 v.set(fields[1]);                                  // 3 输出数据                 context.write(k,   v);          }   }   

（2）第二次处理，编写TwoIndexReducer

   package com.atguigu.mapreduce.index;   import java.io.IOException;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Reducer;   public class TwoIndexReducer extends Reducer<Text, Text,   Text, Text> {              @Override          protected void   reduce(Text key, Iterable<Text> values, Context context) throws   IOException, InterruptedException {                 //   atguigu a.txt 3                 //   atguigu b.txt 2                 //   atguigu c.txt 2                     //   atguigu c.txt-->2 b.txt-->2 a.txt-->3                     StringBuilder   sb = new StringBuilder();                     for   (Text value : values) {                        sb.append(value.toString().replace("\t",   "-->") + "\t");                 }                                  context.write(key,   new Text(sb.toString()));          }   }   

（3）第二次处理，编写TwoIndexDriver

   package com.atguigu.mapreduce.index;   import org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class TwoIndexDriver {              public static   void main(String[] args) throws Exception {                 Configuration   config = new Configuration();                 Job job   = Job.getInstance(config);                     job.setMapperClass(TwoIndexMapper.class);                 job.setReducerClass(TwoIndexReducer.class);                     job.setMapOutputKeyClass(Text.class);                 job.setMapOutputValueClass(Text.class);                                  job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(Text.class);                     FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                     System.exit(job.waitForCompletion(true)   ? 0 : 1);          }   }   

（4）第二次查看最终结果

   atguigu    c.txt-->2  b.txt-->2 a.txt-->3     pingping  c.txt-->1  b.txt-->3 a.txt-->1     ss     c.txt-->1  b.txt-->1 a.txt-->2     

## 3.9 找微信共同好友分析实战

1）需求：

以下是微信的好友列表数据，冒号前是一个用，冒号后是该用户的所有好友（数据中的好友关系是单向的）

![img](D:\software\Typora\imgs\clip_image096-1544403281343.png)

求出哪些人两两之间有共同好友，及他俩的共同好友都有谁？

2）需求分析：

先求出A、B、C、….等是谁的好友

第一次输出结果

   A     I,K,C,B,G,F,H,O,D,   B     A,F,J,E,   C     A,E,B,H,F,G,K,   D     G,C,K,A,L,F,E,H,   E     G,M,L,H,A,F,B,D,   F     L,M,D,C,G,A,   G     M,   H     O,   I      O,C,   J      O,   K     B,   L     D,E,   M    E,F,   O     A,H,I,J,F,   

第二次输出结果

   A-B E C    A-C D F    A-D E F    A-E D B C    A-F O B C D E    A-G F E C D    A-H E C D O    A-I  O    A-J  O B    A-K D C    A-L F E D    A-M E F    B-C A    B-D A E    B-E C    B-F  E A C    B-G C E A    B-H A E C    B-I  A    B-K C A    B-L E    B-M E    B-O A    C-D A F    C-E D    C-F  D A    C-G D F A    C-H D A    C-I  A    C-K A D    C-L D F    C-M F    C-O I A    D-E L    D-F A E    D-G E A F    D-H A E    D-I  A    D-K A    D-L E F    D-M F E    D-O A    E-F  D M C B    E-G C D    E-H C D    E-J  B    E-K C D    E-L D    F-G D C A E    F-H A D O E C    F-I   O A    F-J   B O    F-K D C A    F-L  E D    F-M E    F-O A    G-H D C E A    G-I  A    G-K D A C    G-L D F E    G-M E F    G-O A    H-I  O A    H-J  O    H-K A C D    H-L D E    H-M E    H-O A    I-J   O    I-K  A    I-O  A    K-L D    K-O A    L-M E F   

3）代码实现：

（1）第一次Mapper 

   package com.atguigu.mapreduce.friends;   import java.io.IOException;   import org.apache.hadoop.io.LongWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Mapper;       public class OneShareFriendsMapper extends   Mapper<LongWritable, Text, Text, Text>{                    @Override          protected void   map(LongWritable key, Text value, Mapper<LongWritable, Text, Text,   Text>.Context context)                        throws   IOException, InterruptedException {                 // 1 获取一行 A:B,C,D,F,E,O                 String   line = value.toString();                                  // 2 切割                 String[]   fileds = line.split(":");                                  // 3 获取person和好友                 String   person = fileds[0];                 String[]   friends = fileds[1].split(",");                                  // 4写出去                 for(String   friend: friends){                        //   输出 <好友，人>                        context.write(new   Text(friend), new Text(person));                 }          }   }   

（2）第一次Reducer 

   package com.atguigu.mapreduce.friends;   import java.io.IOException;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Reducer;       public class OneShareFriendsReducer extends   Reducer<Text, Text, Text, Text>{                    @Override          protected void   reduce(Text key, Iterable<Text> values, Context context)                        throws   IOException, InterruptedException {                                  StringBuffer   sb = new StringBuffer();                 //1 拼接                 for(Text   person: values){                        sb.append(person).append(",");                 }                                  //2 写出                 context.write(key,   new Text(sb.toString()));          }   }   

（3）第一次Driver 

   package com.atguigu.mapreduce.friends;       import org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class OneShareFriendsDriver {              public static void   main(String[] args) throws Exception {                 // 1 获取job对象                 Configuration   configuration = new Configuration();                 Job job   = Job.getInstance(configuration);                                  // 2 指定jar包运行的路径                 job.setJarByClass(OneShareFriendsDriver.class);                     // 3 指定map/reduce使用的类                 job.setMapperClass(OneShareFriendsMapper.class);                 job.setReducerClass(OneShareFriendsReducer.class);                                  // 4 指定map输出的数据类型                 job.setMapOutputKeyClass(Text.class);                 job.setMapOutputValueClass(Text.class);                                  // 5 指定最终输出的数据类型                 job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(Text.class);                                  // 6 指定job的输入原始所在目录                 FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                                  // 7 提交                 boolean   result = job.waitForCompletion(true);                                  System.exit(result?1:0);          }   }   

（4）第二次Mapper 

   package com.atguigu.mapreduce.friends;   import java.io.IOException;   import java.util.Arrays;   import org.apache.hadoop.io.LongWritable;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Mapper;       public class TwoShareFriendsMapper extends   Mapper<LongWritable, Text, Text, Text>{                    @Override          protected void   map(LongWritable key, Text value, Context context)                        throws   IOException, InterruptedException {                 // A   I,K,C,B,G,F,H,O,D,                 // 友 人，人，人                 String line   = value.toString();                 String[]   friend_persons = line.split("\t");                     String   friend = friend_persons[0];                 String[]   persons = friend_persons[1].split(",");                     Arrays.sort(persons);                     for (int   i = 0; i < persons.length - 1; i++) {                                                for   (int j = i + 1; j < persons.length; j++) {                               //   发出 <人-人，好友> ，这样，相同的“人-人”对的所有好友就会到同1个reduce中去                               context.write(new   Text(persons[i] + "-" + persons[j]), new Text(friend));                        }                 }          }   }   

（5）第二次Reducer 

   package com.atguigu.mapreduce.friends;   import java.io.IOException;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Reducer;       public class TwoShareFriendsReducer extends   Reducer<Text, Text, Text, Text>{                    @Override          protected void   reduce(Text key, Iterable<Text> values, Context context)                        throws   IOException, InterruptedException {                                  StringBuffer   sb = new StringBuffer();                     for   (Text friend : values) {                        sb.append(friend).append("   ");                 }                                  context.write(key,   new Text(sb.toString()));          }   }   

（6）第二次Driver 

   package com.atguigu.mapreduce.friends;       import org.apache.hadoop.conf.Configuration;   import org.apache.hadoop.fs.Path;   import org.apache.hadoop.io.Text;   import org.apache.hadoop.mapreduce.Job;   import   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       public class TwoShareFriendsDriver {              public static   void main(String[] args) throws Exception {                 // 1 获取job对象                 Configuration   configuration = new Configuration();                 Job job   = Job.getInstance(configuration);                                  // 2 指定jar包运行的路径                 job.setJarByClass(TwoShareFriendsDriver.class);                     // 3 指定map/reduce使用的类                 job.setMapperClass(TwoShareFriendsMapper.class);                 job.setReducerClass(TwoShareFriendsReducer.class);                                  // 4 指定map输出的数据类型                 job.setMapOutputKeyClass(Text.class);                 job.setMapOutputValueClass(Text.class);                                  // 5 指定最终输出的数据类型                 job.setOutputKeyClass(Text.class);                 job.setOutputValueClass(Text.class);                                  // 6 指定job的输入原始所在目录                 FileInputFormat.setInputPaths(job,   new Path(args[0]));                 FileOutputFormat.setOutputPath(job,   new Path(args[1]));                                  // 7 提交                 boolean   result = job.waitForCompletion(true);                                  System.exit(result?1:0);          }   }   

## 3.10 压缩/解压缩

### 3.10.1 对数据流的压缩和解压缩

CompressionCodec有两个方法可以用于轻松地压缩或解压缩数据。要想对正在被写入一个输出流的数据进行压缩，我们可以使用 createOutputStream(OutputStreamout)方法创建一个CompressionOutputStream(未压缩的数据将 被写到此)，将其以压缩格式写入底层的流。相反，要想对从输入流读取而来的数据进行解压缩，则调用 createInputStream(InputStreamin)函数，从而获得一个CompressionInputStream,，从而从底层的流 读取未压缩的数据。

测试一下如下压缩方式：

| DEFLATE | org.apache.hadoop.io.compress.DefaultCodec |
| ------- | ------------------------------------------ |
| gzip    | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2   | org.apache.hadoop.io.compress.BZip2Codec   |
| LZ4     | org.apache.hadoop.io.compress.Lz4Codec     |

 

   **package**   com.atguigu.mapreduce.compress;   **import**   java.io.File;   **import**   java.io.FileInputStream;   **import**   java.io.FileNotFoundException;   **import**   java.io.FileOutputStream;   **import**   java.io.IOException;   **import**   java.io.InputStream;   **import**   java.io.OutputStream;   **import**   org.apache.hadoop.conf.Configuration;   **import**   org.apache.hadoop.fs.Path;   **import**   org.apache.hadoop.io.IOUtils;   **import**   org.apache.hadoop.io.compress.CompressionCodec;   **import**   org.apache.hadoop.io.compress.CompressionCodecFactory;   **import**   org.apache.hadoop.io.compress.CompressionOutputStream;   **import**   org.apache.hadoop.util.ReflectionUtils;       **public** **class**   TestCompress {                    **public**   **static** **void** main(String[] args) **throws** Exception,   IOException {   //            compress("e:/test.txt","org.apache.hadoop.io.compress.BZip2Codec");                 *decompres*("e:/test.txt.bz2");          }                    /*           * 压缩           * filername：要压缩文件的路径           * method：欲使用的压缩的方法（org.apache.hadoop.io.compress.BZip2Codec）           */          **public**   **static** **void** compress(String filername, String method) **throws**   ClassNotFoundException, IOException {                                  //   1 创建压缩文件路径的输入流                 File   fileIn = **new** File(filername);                 InputStream   in = **new** FileInputStream(fileIn);                                  //   2 获取压缩的方式的类                 Class   codecClass = Class.*forName*(method);                                  Configuration   conf = **new** Configuration();                 //   3 通过名称找到对应的编码/解码器                 CompressionCodec   codec = (CompressionCodec) ReflectionUtils.*newInstance*(codecClass,   conf);                     //   4 该压缩方法对应的文件扩展名                 File   fileOut = **new** File(filername + codec.getDefaultExtension());                 fileOut.delete();                     OutputStream   out = **new** FileOutputStream(fileOut);                 CompressionOutputStream   cout = codec.createOutputStream(out);                     //   5 流对接                 IOUtils.*copyBytes*(in,   cout, 1024 * 1024 * 5, **false**); // 缓冲区设为5MB                     //   6 关闭资源                 in.close();                 cout.close();          }              /*           * 解压缩           * filename：希望解压的文件路径           */          **public**   **static** **void** decompres(String filename) **throws**   FileNotFoundException, IOException {                     Configuration   conf = **new** Configuration();                 CompressionCodecFactory   factory = **new** CompressionCodecFactory(conf);                                  //   1 获取文件的压缩方法                 CompressionCodec   codec = factory.getCodec(**new** Path(filename));                                  //   2 判断该压缩方法是否存在                 **if**   (**null** == codec) {                        System.**out**.println("Cannot   find codec for file " + filename);                        **return**;                 }                     //   3 创建压缩文件的输入流                 InputStream   cin = codec.createInputStream(**new** FileInputStream(filename));                                  //   4 创建解压缩文件的输出流                 File   fout = **new** File(filename + ".decoded");                 OutputStream   out = **new** FileOutputStream(fout);                     //   5 流对接                 IOUtils.*copyBytes*(cin,   out, 1024 * 1024 * 5, **false**);                     //   6 关闭资源                 cin.close();                 out.close();          }   }   

### 3.10.2 在Map输出端采用压缩

即使你的MapReduce的输入输出文件都是未压缩的文件，你仍然可以对map任务的中间结果输出做压缩，因为它要写在硬盘并且通过网络传输到reduce节点，对其压缩可以提高很多性能，这些工作只要设置两个属性即可，我们来看下代码怎么设置：

   **package**   com.atguigu.mapreduce.compress;   **import**   java.io.IOException;   **import**   org.apache.hadoop.conf.Configuration;   **import**   org.apache.hadoop.fs.Path;   **import**   org.apache.hadoop.io.IntWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.io.compress.BZip2Codec;   **import**   org.apache.hadoop.io.compress.CompressionCodec;   **import**   org.apache.hadoop.io.compress.GzipCodec;   **import**   org.apache.hadoop.mapreduce.Job;   **import**   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import**   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       **public** **class**   WordCountDriver {              **public**   **static** **void** main(String[] args) **throws** IOException,   ClassNotFoundException, InterruptedException {                     Configuration   configuration = **new** Configuration();                     // 开启map端输出压缩                 configuration.setBoolean("mapreduce.map.output.compress",   **true**);                 // 设置map端输出压缩方式                 configuration.setClass("mapreduce.map.output.compress.codec",   GzipCodec.**class**, CompressionCodec.**class**);                     Job   job = Job.*getInstance*(configuration);                     job.setJarByClass(WordCountDriver.**class**);                     job.setMapperClass(WordCountMapper.**class**);                 job.setReducerClass(WordCountReducer.**class**);                     job.setMapOutputKeyClass(Text.**class**);                 job.setMapOutputValueClass(IntWritable.**class**);                     job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(IntWritable.**class**);                     FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                     **boolean**   result = job.waitForCompletion(**true**);                     System.*exit*(result   ? 1 : 0);          }   }   

2）Mapper保持不变

   **package**   com.atguigu.mapreduce.compress;   **import**   java.io.IOException;   **import**   org.apache.hadoop.io.IntWritable;   **import** org.apache.hadoop.io.LongWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Mapper;       **public** **class**   WordCountMapper **extends** Mapper<LongWritable, Text, Text,   IntWritable>{                    @Override          **protected**   **void** map(LongWritable key, Text value, Context context)                        **throws**   IOException, InterruptedException {                                  String   line = value.toString();                                  String[]   words = line.split(" ");                                  **for**(String   word:words){                        context.write(**new**   Text(word), **new** IntWritable(1));                 }          }   }   

3）Reducer保持不变

   **package**   com.atguigu.mapreduce.compress;       **import**   java.io.IOException;       **import**   org.apache.hadoop.io.IntWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.mapreduce.Reducer;       **public** **class**   WordCountReducer **extends** Reducer<Text, IntWritable, Text, IntWritable>{                    @Override          **protected**   **void** reduce(Text key, Iterable<IntWritable> values,                        Context   context) **throws** IOException, InterruptedException {                                  **int**   count = 0;                                  **for**(IntWritable   value:values){                        count   += value.get();                 }                                  context.write(key,   **new** IntWritable(count));          }   }   

### 3.10.3 在Reduce输出端采用压缩

基于workcount案例处理

1）修改驱动

   **package**   com.atguigu.mapreduce.compress;   **import**   java.io.IOException;   **import**   org.apache.hadoop.conf.Configuration;   **import**   org.apache.hadoop.fs.Path;   **import**   org.apache.hadoop.io.IntWritable;   **import**   org.apache.hadoop.io.Text;   **import**   org.apache.hadoop.io.compress.BZip2Codec;   **import** org.apache.hadoop.io.compress.DefaultCodec;   **import** org.apache.hadoop.io.compress.GzipCodec;   **import** org.apache.hadoop.io.compress.Lz4Codec;   **import** org.apache.hadoop.io.compress.SnappyCodec;   **import**   org.apache.hadoop.mapreduce.Job;   **import**   org.apache.hadoop.mapreduce.lib.input.FileInputFormat;   **import**   org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;       **public** **class**   WordCountDriver {              **public**   **static** **void** main(String[] args) **throws** IOException,   ClassNotFoundException, InterruptedException {                                  Configuration   configuration = **new** Configuration();                                  Job   job = Job.*getInstance*(configuration);                                  job.setJarByClass(WordCountDriver.**class**);                                  job.setMapperClass(WordCountMapper.**class**);                 job.setReducerClass(WordCountReducer.**class**);                                  job.setMapOutputKeyClass(Text.**class**);                 job.setMapOutputValueClass(IntWritable.**class**);                                  job.setOutputKeyClass(Text.**class**);                 job.setOutputValueClass(IntWritable.**class**);                                  FileInputFormat.*setInputPaths*(job,   **new** Path(args[0]));                 FileOutputFormat.*setOutputPath*(job,   **new** Path(args[1]));                                  // 设置reduce端输出压缩开启                 FileOutputFormat.*setCompressOutput*(job, **true**);                                  // 设置压缩的方式              FileOutputFormat.*setOutputCompressorClass*(job,   BZip2Codec.**class**);    //           FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class);    //           FileOutputFormat.setOutputCompressorClass(job, Lz4Codec.class);    //           FileOutputFormat.setOutputCompressorClass(job, DefaultCodec.class);                                **boolean**   result = job.waitForCompletion(**true**);                                  System.*exit*(result?1:0);          }   }   

2）Mapper和Reducer保持不变（详见3.10.2）

# 四、常见错误

1）导包容易出错。尤其Text.

2）Mapper中第一个输入的参数必须是LongWritable或者NullWritable，不可以是IntWritable.  报的错误是类型转换异常。

3）java.lang.Exception: java.io.IOException: Illegal partition for 13926435656 (4)，说明partition和reducetask个数没对上，调整reducetask个数。

4）如果分区数不是1，但是reducetask为1，是否执行分区过程。答案是：不执行分区过程。因为在maptask的源码中，执行分区的前提是先判断reduceNum个数是否大于1。不大于1肯定不执行。

 