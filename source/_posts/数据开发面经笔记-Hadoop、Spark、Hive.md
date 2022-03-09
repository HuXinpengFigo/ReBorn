---
title: 数据开发面经笔记-Hadoop、Spark、Hive
date: 2022-03-08 22:01:38
index_img: /img/Apache_Spark_logo.svg.png
banner_img: /img/hadoop-vs-spark-main-1280x720.jpeg
tags: [Spark,Hadoop,Hive,大数据,面经]
categories: 实习
---

## Hadoop

### Hadoop组件

> Hadoop集群具体来说包含两个集群：HDFS集群和YARN集群，两者逻辑上分离，但物理上常在一起。
>
> （1）HDFS集群：负责海量数据的存储，集群中的角色主要有 NameNode / DataNode/SecondaryNameNode。
>
> （2）YARN集群：负责海量数据运算时的资源调度，集群中的角色主要有 ResourceManager /NodeManager
>
> （3）MapReduce：它其实是一个应用程序开发包。

#### HDFS（Hadoop分布式文件系统）

> 是hadoop体系中数据存储管理的基础。他是一个高度容错的系统，能检测和应对硬件故障。
>
> * client：切分文件，访问HDFS，与namenode交互，获取文件位置信息，与DataNode交互，读取和写入数据。
> * namenode：master节点，在hadoop1.x中只有一个，管理HDFS的名称空间和数据块映射信息，配置副本策略，处理客户 端请求。
> * DataNode：slave节点，存储实际的数据，汇报存储信息给namenode。
> * secondary namenode：辅助namenode，分担其工作量：定期合并fsimage和fsedits，推送给namenode；紧急情况下和辅助恢复namenode，但其并非namenode的热备。

#### MapReduce（分布式计算框架）

> mapreduce是一种计算模型，用于处理大数据量的计算。其中map对应数据集上的独立元素进行指定的操作，生成键-值对形式中间，reduce则对中间结果中相同的键的所有值进行规约，以得到最终结果。
>
> * jobtracker：master节点，只有一个，管理所有作业，任务/作业的监控，错误处理等，将任务分解成一系列任务，并分派给tasktracker。
>
> * tacktracker：slave节点，运行 map task和reducetask；并与jobtracker交互，汇报任务状态。
> * map task：解析每条数据记录，传递给用户编写的map（）并执行，将输出结果写入到本地磁盘（如果为map—only作业，则直接写入HDFS）。
> * reduce task：从map 它深刻地执行结果中，远程读取输入数据，对数据进行排序，将数据分组传递给用户编写的reduce函数执行。

#### **Hive（基于hadoop的数据仓库）**

>  由Facebook开源，最初用于解决海量结构化的日志数据统计问题。
>
> Hive定于了一种类似sql的查询语言（HQL）将SQL转化为mapreduce任务在hadoop上执行。

#### HBase（分布式列存数据库）

> HBase是一个针对结构化数据的可伸缩，高可靠，高性能，分布式和面向列的动态模式数据库。和传统关系型数据库不同，HBase采用了bigtable的数据模型：增强了稀疏排序映射表（key/value）。其中，键由行关键字，列关键字和时间戳构成，HBase提供了对大规模数据的随机，实时读写访问，同时，HBase中保存的数据可以使用MapReduce来处理，它将数据存储和并行计算完美结合在一起。

#### **zookeeper（分布式协作服务）**

>  解决分布式环境下的数据管理问题：统一命名，状态同步，集群管理，配置同步等。

#### sqoop（数据同步工具）

>  sqoop是sql-to-hadoop的缩写，主要用于传统数据库和hadoop之间传输数据。
>
> 数据的导入和导出本质上是mapreduce程序，充分利用了MR的并行化和容错性。

#### pig（基于Hadoop的数据流系统）

> 定义了一种数据流语言**pig latin**，将脚本转换为mapreduce任务在hadoop上执行。
>
> 通常用于离线分析。

参考： [hadoop三大核心组件](https://www.cnblogs.com/jiataoq/p/9700469.html)

### HDFS读流程和写流程

#### HDFS读数据流程

> （1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
>
> （2）NameNode返回是否可以上传。
>
> （3）客户端请求第一个 Block上传到哪几个DataNode服务器上。
>
> （4）NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。
>
> （5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。
>
> （6）dn1、dn2、dn3逐级应答客户端。
>
> （7）客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。
>
> （8）当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。

#### HDFS写数据流程

> （1）客户端通过Distributed FileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。
>
> （2）挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。
>
> （3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。
>
> （4）客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。

### HDFS小文件问题（Block大小设置）

> 在Hadoop2.x之后的版本中，文件块的默认大小是128M，老版本中默认是64M；
>
> Block 大小设置原则：最小化寻址开销,减少网络传输。
>
> * 减少硬盘寻道时间(disk seek time)：HDFS的设计是为了支持大数据操作，合适的block大小有助于减少硬盘寻道时间（平衡了硬盘寻道时间、IO时间），提高系统吞吐量。
>
> * 减少NameNode内存消耗：NameNode需要在内存FSImage文件中记录DataNode中数据块信息，若block size太小，那么需要维护的数据块信息会更多。而HDFS只有一个NameNode节点，内存是极其有限的。
>
> * map崩溃问题：若Map任务崩溃，重新启动执行需要重新加载数据，数据块越大，数据加载时间将越长，恢复时间越长。
>
> * 监管时间问题：主节点监管其他节点的情况，每个节点会周期性的把完成的工作和状态的更新报告回来。若某个节点保存沉默超过预设的时间间隔，主节点“宣告”该节点状态为死亡，并把分配给这个节点的数据发到别的节点。预设的时间间隔是根据数据块 size角度估算的，若size设置不合理，容易误判节点死亡。
>
> * 约束Map任务输出：MapReduce框架中Map任务输出的结果是要经过排序才给reduce函数操作的。在Map任务的merge on disk和Reduce任务中合并溢写生的文件，用到归并排序算法，对小文件进行排序，然后将小文件归并成大文件。
>
> * 网络传输问题: 在数据读写计算的时候,需要进行网络传输.如果block过大会导致网络传输时间增长,程序卡顿/超时/无响应. 任务执行的过程中拉取其他节点的block或者失败重试的成本会过高.如果block过小,则会频繁的进行文件传输,对严重占用网络/CPU资源.

#### 为什么Hadoop中block大小为128MB

> 1. HDFS中平均寻址时间大概为10ms；
> 2.  经过测试发现，寻址时间为传输时间的1%时，为最佳状态；所以最佳传输时间为10ms/0.01=1000ms=1s
> 3. 目前磁盘的传输速率普遍为100MB/s , 网卡普遍为千兆网卡传输速率普遍也是100MB/s；计算出最佳block大小：100MB/s x 1s = 100MB
>
> 所以我们设定block大小为128MB。

#### HDFS中有很多小文件

> 小文件是指文件大小明显小于HDFS上块（block）大小（默认128MB）的文件。如果存储小文件，必定会有大量这样的小文件，否则你也不会使用[Hadoop](https://so.csdn.net/so/search?q=Hadoop&spm=1001.2101.3001.7020)（If you’re storing small files, then you probably have lots of them (otherwise you wouldn’t turn to Hadoop)），这样的文件给hadoop的扩展性和性能带来严重问题。当一个文件的大小小于HDFS的块大小（默认128MB），就将认定为**小文件**否则就是大文件。
>
> 
>
> 至少有两种**场景**下会产生大量的小文件：
>
> （1）这些小文件都是一个大逻辑文件的一部分。由于HDFS在2.x版本开始支持对文件的append，所以在此之前保存无边界文件（例如，log文件）（译者注：持续产生的文件，例如日志每天都会生成）一种常用的方式就是将这些数据以块的形式写入HDFS中（a very common pattern for saving unbounded files (e.g. log files) is to write them in chunks into HDFS）。
>
> （2）文件本身就是很小。设想一下，我们有一个很大的图片语料库，每一个图片都是一个独一的文件，并且没有一种很好的方法来将这些文件合并为一个大的文件。
>
> 这两种情况需要有不同的解决方 式。
>
> 1. 第一种情况
>
>    对于第一种情况，文件是许多记录（Records）组成的，那么可以通过调用HDFS的sync()方法(和append方法结合使用)，每隔一定时间生成一个大文件。或者，可以通过写一个程序来来合并这些小文件（可以看一下Nathan Marz关于Consolidator一种小工具的文章）。
>
> 2. 第二种情况
>
>    对于第二种情况，就需要某种形式的容器通过某种方式来对这些文件进行分组。
>
>    1.  HAR File
>
>       Hadoop Archives （HAR files）是在0.18.0版本中引入到HDFS中的，它的出现就是为了缓解大量小文件消耗NameNode内存的问题。HAR文件是通过在HDFS上构建一个分层文件系统来工作。HAR文件通过hadoop archive命令来创建，而这个命令实 际上是运行了一个MapReduce作业来将小文件打包成少量的HDFS文件（译者注：将小文件进行合并几个大文件）。对于client端来说，使用HAR文件没有任何的改变：所有的原始文件都可见以及可访问（只是使用har://URL，而不是hdfs://URL），但是在HDFS中中文件数却减少了。
>
>    2. SequenceFile
>
>       通常对于"小文件问题"的回应会是：使用序列文件（SequenceFile）。这种方法的思路是，使用文件名（filename）作为key，并且文件内容（file contents）作为value，如下图。在实践中这种方式非常有效。我们回到10,000个100KB小文件问题上，你可以编写一个程序将它们放入一个单一的SequenceFile，然后你可以流式处理它们（直接处理或使用MapReduce）操作SequenceFile。
>
>       ![image](/Users/figo/fluid/themes/fluid/source/img/0a6e3ff0d40e42bc581c88b6cada856825ea325br1-512-512v2_uhq.png)
>
>    3. HBase
>
>       如果你生产很多小文件，那么根据访问模式，不同类型的存储可能更合适（If you are producing lots of small files, then, depending on the access pattern, a different type of storage might be more appropriate）。HBase以Map Files（带索引的SequenceFile）方式存储数据，如果您需要随机访问来执行MapReduce式流式分析，这是一个不错的选择.

#### NameNode和Secondary NameNode工作机制

##### NameNode的结构

> ###### Fsimage
>
> Fsimage文件是HDFS文件系统元数据的一个永久性检查点，其中包含HDFS文件系统的所有目录和文件inode的序列化信息。
>
> ###### Edits文件
>
> 存放HDFS文件系统的所有更新操作的逻辑，文件系统客户端执行的所有写操作首先会记录大Edits文件中。
>
> ###### Seen_txid
>
> 文件保存是一个数字，就是最后一个edits_的数字。

##### NameNode的工作机制

> ###### 第一阶段：NameNode启动
>
> （1）第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
>
> （2）客户端对元数据进行增删改的请求。
>
> （3）NameNode记录操作日志，更新滚动日志。
>
> （4）NameNode在内存中对元数据进行增删改。
>
> ###### 第二阶段：Secondary NameNode工作
>
> （1）Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。
>
> （2）Secondary NameNode请求执行CheckPoint。
>
> （3）NameNode滚动正在写的Edits日志。
>
> （4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。
>
> （5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
>
> （6）生成新的镜像文件fsimage.chkpoint。
>
> （7）拷贝fsimage.chkpoint到NameNode。
>
> （8）NameNode将fsimage.chkpoint重新命名成fsimage。

#### DataNode

##### DataNode工作机制

> （1）一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
>
> （2）DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。
>
> （3）心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。
>
> （4）集群运行中可以安全加入和退出一些机器。

##### DataNode数据损坏

> （1）当DataNode读取Block的时候，它会计算CheckSum。
>
> （2）如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
>
> （3）Client读取其他DataNode上的Block。
>
> （4）DataNode在其文件创建后周期验证CheckSum。

### MapReduce

#### MapReduce工作流程

> （1）Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
>
> （2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
>
> （3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
>
> （4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地[排序]()，并在必要时对数据进行合并、压缩等操作。
>
> ```sh
> 溢写阶段详情：
> 步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。
> 步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
> 步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。
> ```
>
> （5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。

#### ReduceTask工作流程

> （1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
>
> （2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
>
> （3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于[排序]()的策略。由于各个MapTask已经实现对自己的处理结果进行了局部[排序]()，因此，ReduceTask只需对所有数据进行一次归并[排序]()即可。
>
> （4）Reduce阶段：reduce()函数将计算结果写到HDFS上。

### Yarn工作机制（作业提交全过程）

> **Yarn** 是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式 的操作系统平台，而 MapReduce 等运算程序则相当于运行于操作系统之上的应用程序。
>
> YARN 主要由 **ResourceManager**、**NodeManager**、**ApplicationMaster** 和 **Container** 等组件
>
> 构成。

#### 1.作业提交

> （1）Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。
>
> （2）Client向RM申请一个作业id。 
>
> （3）RM给Client返回该job资源的提交路径和作业id。
>
> （4）Client提交jar包、切片信息和配置文件到指定的资源提交路径。
>
> （5）Client提交完资源后，向RM申请运行MrAppMaster。

#### 2.作业初始化

> （6）当RM收到Client的请求后，将该job添加到容量调度器中。
>
> （7）某一个空闲的NM领取到该Job。
>
> （8）该NM创建Container，并产生MRAppmaster。
>
> （9）下载Client提交的资源到本地。

#### 3.任务分配

> （10）MrAppMaster向RM申请运行多个MapTask任务资源。
>
> （11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。

#### 4.任务运行

> （12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区[排序]()。
>
> （13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
>
> （14）ReduceTask向MapTask获取相应分区的数据。
>
> （15）程序运行完毕后，MR会向RM申请注销自己。

#### 5.进度和状态更新

> YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。

#### 6.作业完成

> 除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

### Shuffle过程

> Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle
>
> 参考上面：MapReduce工作流程

#### Map阶段优化

> （1）增大环形缓冲区大小。由100m扩大到200m
> （2）增大环形缓冲区溢写的比例。由80%扩大到90%
> （3）减少对溢写文件的merge次数。（10个文件，一次20个merge）
> （4）不影响实际业务的前提下，采用Combiner提前合并，减少 I/O。

#### Reduce阶段优化

> （1）合理设置Map和Reduce数：两个都不能设置太少，也不能设置太多。太少，会导致Task等待，延长处理时间；太多，会导致 Map、Reduce任务间竞争资源，造成处理超时等错误。
> （2）设置Map、Reduce共存：调整slowstart.completedmaps参数，使Map运行到一定程度后，Reduce也开始运行，减少Reduce的等待时间。
> （3）规避使用Reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。
> （4）增加每个Reduce去Map中拿数据的并行数
> （5）集群性能可以的前提下，增大Reduce端存储数据内存的大小。 

#### IO传输

> 采用数据压缩的方式，减少网络IO的的时间。安装Snappy和LZOP压缩编码器。
> 压缩：
> （1）map输入端主要考虑数据量大小和切片，支持切片的有Bzip2、LZO。注意：LZO要想支持切片必须创建索引；
> （2）map输出端主要考虑速度，速度快的snappy、LZO；
> （3）reduce输出端主要看具体需求，例如作为下一个mr输入需要考虑切片，永久保存考虑压缩率比较大的gzip。

### Hadoop的参数

#### 资源相关参数

| 配置参数                                      | 参数说明                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| mapreduce.map.memory.mb                       | 一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.reduce.memory.mb                    | 一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。 |
| mapreduce.map.cpu.vcores                      | 每个MapTask可使用的最多cpu core数目，默认值: 1               |
| mapreduce.reduce.shuffle.parallelcopies       | 每个ReduceTask可使用的最多cpu core数目，默认值: 1            |
| mapreduce.reduce.shuffle.merge.percent        | 每个Reduce去Map中取数据的并行数。默认值是5                   |
| mapreduce.reduce.shuffle.input.buffer.percent | Buffer中的数据达到多少比例开始写入磁盘。默认值0.66           |
| mapreduce.reduce.input.buffer.percent         | Buffer大小占Reduce可用内存的比例。默认值0.7 指定多少比例的内存用来存放Buffer中的数据，默认值是0.0 |

#### YARN

| 配置参数                                 | 参数说明                                        |
| ---------------------------------------- | ----------------------------------------------- |
| yarn.scheduler.minimum-allocation-mb     | 给应用程序Container分配的最小内存，默认值：1024 |
| yarn.scheduler.maximum-allocation-mb     | 给应用程序Container分配的最大内存，默认值：8192 |
| yarn.scheduler.minimum-allocation-vcores | 每个Container申请的最小CPU核数，默认值：1       |
| yarn.scheduler.maximum-allocation-vcores | 每个Container申请的最大CPU核数，默认值：32      |
| yarn.nodemanager.resource.memory-mb      | 给Containers分配的最大物理内存，默认值：8192    |

#### Shuffle

| 配置参数                         | 参数说明                          |
| -------------------------------- | --------------------------------- |
| mapreduce.task.io.sort.mb        | Shuffle的环形缓冲区大小，默认100m |
| mapreduce.map.sort.spill.percent | 环形缓冲区溢出的阈值，默认80%     |

#### 容错相关参数

| 配置参数                     | 参数说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| mapreduce.map.maxattempts    | 每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.reduce.maxattempts | 每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。 |
| mapreduce.task.timeout       | Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大。 |

### 异构存储（冷热数据分离）

> 所谓的异构存储就是将不同需求或者冷热的数据存储到不同的介质中去，实现既能兼顾性能又能兼顾成本。

#### 存储类型

> HDFS异构存储支持如下4种类型，分别是：
>
> - RAM_DISK（内存镜像文件系统） 
> - SSD（固态硬盘） 
> - DISK（普通磁盘，HDFS默认） 
> - ARCHIVE（计算能力弱而存储密度高的存储介质，一般用于归档） 
>
> 以上四种自上到下，速度由快到慢，单位存储成本由高到低。

#### 存储策略

HDFS总共支持**Lazy_Persist**、**All_SSD**、**One_SSD**、**Hot**、**Warm**和**Cold**等6种存储策略。

| 策略         | 说明                                                  |
| ------------ | ----------------------------------------------------- |
| Lazy_Persist | 1份数据存储在[RAM_DISK]即内存中，其他副本存储在DISK中 |
| All_SSD      | 全部数据都存储在SSD中                                 |
| One_SSD      | 一份数据存储在SSD中，其他副本存储在DISK中             |
| Hot          | 全部数据存储在DISK中，默认策略为Hot                   |
| Warm         | 一份数据存储在DISK中，其他数据存储方式为ARCHIVE       |
| Cold         | 全部数据以ARCHIVE的方式保存                           |

## hive

### Hive 工作原理

> Hive构建在Hadoop之上
>
> 1、HQL中对查询语句的解释、优化、生成查询计划是由Hive完成的
>
> 2、所有的数据都是存储在Hadoop中
>
> 3、查询计划被转化为MapReduce任务，在Hadoop中执行（有些查询没有MR任务，如：select * from table）
>
> 4、Hadoop和Hive都是用UTF-8编码的

### Hive 数据类型&文件格式

> Hive 提供了基本数据类型和复杂数据类型

#### 原始数据类型

- 整型

  - **TINYINT** — 微整型，只占用1个字节，只能存储0-255的整数。
  - **SMALLINT**– 小整型，占用2个字节，存储范围–32768 到 32767。
  - **INT**– 整型，占用4个字节，存储范围-2147483648到2147483647。
  - **BIGINT**– 长整型，占用8个字节，存储范围-2^63到2^63-1。

- 布尔型

  - **BOOLEAN** — TRUE/FALSE

- 浮点型

  - **FLOAT**– 单精度浮点数。
  - **DOUBLE**– 双精度浮点数。

- 字符串型

- - **STRING**– 不设定长度。

#### 复合数据类型

- Structs：一组由任意数据类型组成的结构。比如，定义一个字段C的类型为STRUCT {a INT; b STRING}，则可以使用a和C.b来获取其中的元素值；
- Maps：和Java中的Map相同，即存储K-V对的；
- Arrays：数组；

![img](/Users/figo/fluid/themes/fluid/source/img/778187-20180115161529224-1291199969.png)

#### Hive文件格式

```sql
TEXTFILE //文本，默认值

SEQUENCEFILE // 二进制序列文件

RCFILE //列式存储格式文件 Hive0.6以后开始支持

ORC //列式存储格式文件，比RCFILE有更高的压缩比和读写效率，Hive0.11以后开始支持

PARQUET //列出存储格式文件，Hive0.13以后开始支持
```

### Hive Sort by与Order by的区别

> order by实现的是全局排序，在hive mr引擎中将会只有1个**reduce**。而使用sort by会起多个reduce，只会在每个reduce中排序，如果不指定分组的话，跑出来的数据看起来是杂乱无章的，如果指定reduce个数是1，那么结果和order by是一致的。
>
> 
>
> order by一般配合group by使用，而group by需要配合[聚合函数](https://so.csdn.net/so/search?q=聚合函数&spm=1001.2101.3001.7020)使用，举个例子：
>
> ![img](/Users/figo/fluid/themes/fluid/source/img/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xTQjE5OTMwNzA2,size_16,color_FFFFFF,t_70.png)
>
> 而sort by分组时需要使用distribute by，和group by类似，但是它不需要配合聚合函数使用，也就不影响原数据的函数，这点和开窗函数有点类似，如下：
>
> ![img](/Users/figo/fluid/themes/fluid/source/img/distributed_by.png)
>
> distribute by还有个简化版，当distribute by和sort by的字段相同时，可以简写为cluster by。
>
> 
>
> 总结：order by是**全局**排序，sort by是组内排序。distribute by sort by可以结合桶表使用，给桶中的数据排序。

### Hive优化

推荐：[Hive学习之路 （二十一）Hive 优化策略](https://www.cnblogs.com/qingyunzong/p/8847775.html#_label1)

> 1、好的模型设计事半功倍
>
> 2、解决数据倾斜问题
>
> 3、减少 job 数
>
> 4、设置合理的 MapReduce 的 task 数，能有效提升性能。(比如，10w+级别的计算，用 160个 reduce，那是相当的浪费，1 个足够)
>
> 5、了解数据分布，自己动手解决数据倾斜问题是个不错的选择。这是通用的算法优化，但 算法优化有时不能适应特定业务背景，开发人员了解业务，了解数据，可以通过业务逻辑精 确有效的解决数据倾斜问题
>
> 6、数据量较大的情况下，慎用 count(distinct)，group by 容易产生倾斜问题
>
> 7、对小文件进行合并，是行之有效的提高调度效率的方法，假如所有的作业设置合理的文 件数，对云梯的整体调度效率也会产生积极的正向影响
>
> 8、优化时把握整体，单个作业最优不如整体最优

### Hive分桶

#### 什么是分桶？和分区有什么区别？

> 分区：Hive在查询数据的时候，一般会扫描整个表的数据,会消耗很多不必要的时间。有些时候，我们只需要关心一部分数据,比如WHERE子句的查询条件，那这时候这种全表扫描的方式是很影响性能的。从而引入了分区的概念。分区就是对数据进行分类，这样在查询的时候，就可以只是针对分区查询，从而不必全表扫描。
>
> 
>
> 一个**目录**对应一个**分区**
>
> 
>
> 分桶：并非所有的数据集都可形成合理的分区，特别之前所提到过的要确定合适的划分大小的疑虑。对于每一个表或者分区，可以进一步细分成桶，桶是对数据进行更细粒度的划分。Hive默认采用对某一列的每个数据进行hash（哈希），使用hashcode对 桶的个数求余，确定该条记录放入哪个桶中。
>
> 分桶实际上和 MapReduce中的分区是一样的。分桶数和reduce数对应。
>
> 
>
> 一个**文件**对应一个**分桶**

