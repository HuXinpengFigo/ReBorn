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
>       ![image](/img/0a6e3ff0d40e42bc581c88b6cada856825ea325br1-512-512v2_uhq.png)
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

![img](/img/778187-20180115161529224-1291199969.png)

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
> ![img](/img/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xTQjE5OTMwNzA2,size_16,color_FFFFFF,t_70.png)
>
> 而sort by分组时需要使用distribute by，和group by类似，但是它不需要配合聚合函数使用，也就不影响原数据的函数，这点和开窗函数有点类似，如下：
>
> ![img](/img/distributed_by.png)
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

### Hive窗口函数

> 有时，想在不影响原有数据粒度形态的前提下，在一个表中实现 得到按照一定规则分析处理后想要的值。
> 于是便于分析的 **窗口函数** 应运而生。
>
> 用 大白话 说就是： 既想要聚合函数作用后的数据结果，
>
> 又想展示聚合前的数据，即不想承担聚合函数带来的数据变为"一行" 的后果。
>
> 这种 **既要又要** 的复杂要求，那就用 **窗口函数** 来满足！

#### 常用场景

```sql
sum(...)over(...)  #常用来多维度分组求和、求累加值等
count(...)over(...)  #常用来多维度分组计数、计算汇总计数等
min、max、avg(...)over(...)  #常用来计算指定分组列对应某指标的最大、最小、平均值
```

下面常用于求分组中涉及[排序](https://www.nowcoder.com/jump/super-jump/word?word=排序)相关场景：

```sql
lag()over(...)
lead()over(...)
row_number()over(...)
rank()over(...)
```

> lag() over() 与 lead() over() 函数是跟偏移量相关的两个[分析](https://so.csdn.net/so/search?q=分析&spm=1001.2101.3001.7020)函数，通过这两个函数**可以在一次查询中取出同一字段的前 N 行的数据 (lag) 和后 N 行的数据 (lead) 作为独立的列, 从而更方便地进行进行数据过滤。**
>
> [row_number() OVER(PARTITION BY)函数介绍](https://blog.csdn.net/631799/article/details/7419797)：求排序的开窗函数，与**rank()OVER()**做对比，rank有并列第一则接下来是第三，row_number并列第一只会显示一个，而**dense_rank()**是连续排序，有两个第二名时仍然跟着第三名。

#### **窗口函数语法**

```sql
# 语法形式申明：
window_func() over (partition by [<col1>,<col2>,…]
[order by <col1>[asc/desc], <col2>[asc/desc]…] <窗口选取语句: windowing_clause>)
```

#### **①** **window_func**:

- 常用聚合函数有： 

  sum()、count()、avg()、max() 

-  常用序列函数有： 

  row_number()、rank()、dense_rank()、first_value()、last_value()、lag()、lead()

#### **② partition by [<col1>,<col2>,…] ：** 

-  指定开窗的列，即用于分组的列名。 
-  可选项。即使不指定也可以正常使用窗口函数，此时将整个结果集当做一个大的窗口来用。 
-  可认为是查询分区子句，较类似group by：都是将数据按照指定的列进行分组，分区列的值相同的行被视为在同一个窗口内。 

####  **③ order by <col1>[asc/desc], <col2>[asc/desc]…：** 

-  可选项。指定数据在一个窗口内如何[排序]()，会让输入的数据强制[排序]()，默认asc升序。  

####  **④ windowing_clause：** 

-  可选项。窗口选取语句，指定了窗口函数作用的范围，可以理解为比partition by更细粒度的划分分组范围。 
- 选取关键词：
  -  通常用rows指定开窗方式 
  -  preceding ：向前 
  -  following：向后 
  -  current row：当前行 
  -  unbounded preceding：第一行 
  -  unbounded following：最后一行 
- 两种使用方式：
  -  rows between x preceding / following and y preceding / following：表示窗口范围是从前或后x行到前或后y行，区间表示法即：[x,y]。 
  -  rows x preceding / following：窗口范围是从前或后第x行到当前行。区间表示法即[-x,0]、[0,x]。 

###  **注意点：** 

####  **①** **窗口函数** 

- ####  不能和同级别的聚合函数一起使用。 

- ####  不能嵌套使用窗口函数和聚合函数。  

####  **② partition by**  

-  与聚合函数group by不同的地方：不会减少表中记录的行数，而group by是对原始数据进行聚合统计，一般只有一条反映统计值的结果（每组返回一条）。 
-  over之前的函数在每一个分组之内进行，若超出分组，函数会重新计算。

####  **③** **order by** 

-  order by默认情况下聚合从起始行到当前行的数据 
-  该子句对于[排序]()类函数是必须的，因为如果数据无序，这些函数的结果没有任何意义。

####  **④ 窗口选取从句** 

-  区间一定从前到后，不能从后到前。 
- 从句缺失时：
  -  order by 指定，窗口从句缺失，则窗口的默认值为range between unbounded preceding and current row，也就是从第一行到当前行； 
  -  order by 和窗口从句如果都缺失，则窗口的默认值为range between unbounded preceding and unbounded following，即从第一行到最后一行。 
-  序列函数不支持窗口选取子句。

 **⑤ 执行顺序** 

-  from -> where -> group by -> having -> select -> **window func** -> order by -> limit 
-  可以理解为窗口函数是将select中的结果数据集当做 输入 再次加工处理。

参考：[讲懂高频Hive：窗口函数（一）](https://www.nowcoder.com/discuss/823406?type=post&order=recall&pos=&page=1&ncTraceId=&channel=-1&source_id=search_post_nctrack&gio_id=D68F5CC4A5088E8311C5FF791A330F3B-1647483296687)

### Hive 存储引擎、执行引擎和优化器

> - 存储方面：textfile、orcfile、rcfile、parquet、sequencefile
> - 执行引擎：mr (MapReduce)、tez、spark
> - 词法解析： calcite、cbo
> - 优化：mapjoin
> - 自定义函数：udf
> - sql语法或自带函数

### Hive各种join之间的关系与使用

> **SQL join 用于根据两个或多个表中的列之间的关系，从这些表中查询数据。**
>
> - JOIN: 如果表中有至少一个匹配，则返回行
> - LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行
> - RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行
> - FULL JOIN: 只要其中一个表中存在匹配，就返回行

#### Hive map join

> MapJoin是Hive的一种优化操作，其适用于**小表JOIN大表**的场景，由于表的JOIN操作是在Map端且在内存进行的，所以其并不需要启动Reduce任务也就不需要经过shuffle阶段，从而能在一定程度上节省资源提高JOIN效率。
>
> （在Hive0.11后，Hive**默认启动**该优化，也就是不在需要显示的使用MAPJOIN标记，其会在必要的时候触发该优化操作将普通JOIN转换成MapJoin）

### Hive (数据仓库) 与数据库的区别

>1. **存储数据位置：**Hive在HDFS中，数据库的数据存储在块设备上或者本地文件系统中。
>
>2. **查询语言：**Hive为了方便一些开发者使用，通过将HQL转为MapReduce查询，但是HQL与常规的SQL语句使用起来还是有不同的地方。
>
>3. **数据更新：**因为Hive是数据仓库，本身性质就是“读多写少”，所以在一些以前的版本中，它并不支持insert into插入数据以及update更新数据的。但是数据库通常使用起来就是需要经常修改以及添加的。
>
>4. **计算引擎：**Hive的架构如下，Hive实际上就是将用户输入的HQL语句转化为MapReduce程序来运行，但是数据库有自己的执行引擎。
>
>   ![img](/img/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNTcxOTAw,size_16,color_FFFFFF,t_70.png)
>
>5. **索引：**在Hive中并没有索引这个概念，当Hive需要访问数据中某些符合特定条件的特定值时，需要对全盘进行暴力检索，所以访问延迟较高，但Hive引入了MapReduce，可以并行处理数据。在数据库中，虽不可以并行处理数据，但是可以在数据中建立索引。因此，对于少量特定条件的数据访问时，数据库延迟低。但是大数据量还是选择Hive。
>
>6. **执行延迟：**Hive中没有索引不说，MapReduce的这个框架本身就具有很高的延迟。相对于数据库来说，延迟比较高。但是当数据量很大，超出了数据库的处理能力范围时，Hive的并行计算就具有很大的优势了。
>
>7. **可扩展性：**由于Hive是建立在Hadoop集群上的，所以它的扩展能力和Hadoop一样。所以可以想象，Hadoop集群的节点可以有多少个？但是数据库本身由于ACID语义的限制，就Oracle在理论上最多也只能扩展100台。

参考：[Hive（数据仓库）与数据库的区别](https://blog.csdn.net/qq_41571900/article/details/84640136)

## SPARK

> Spark是一个基于内存的，用于大规模数据处理（离线计算、实时计算、快速查询（交互式查询））的统一分析引擎。它内部的组成模块，包含SparkCore，SparkSQL，SparkStreaming，SparkMLlib，SparkGraghx等。
>
> 
>
> **Spark特点：**
>
> * **快：**Spark计算速度是MapReduce计算速度的10-100倍；
> * **易用：**MR支持1种计算模型，Spsark支持更多的计算模型(算法多)；
> * **通用：**Spark 能够进行离线计算、交互式查询（快速查询）、实时计算、机器学习、图计算；
> * **兼容性：**Spark支持大数据中的Yarn调度，支持mesos。可以处理hadoop计算的数据。

![image-20220317213839838](/img/image-20220317213839838.png)

### Spark基本概念

> * Application：表示你的应用程序
> * Driver：表示main()函数，创建SparkContext。由SparkContext负责与ClusterManager通信，进行资源的申请，任务的分配和监控等。程序执行完毕后关闭SparkContext
>
> * Executor：某个Application运行在Worker节点上的一个进程，该进程负责运行某些task，并且负责将数据存在内存或者磁盘上。在Spark on Yarn模式下，其进程名称为 CoarseGrainedExecutor Backend，一个CoarseGrainedExecutor Backend进程有且仅有一个executor对象，它负责将Task包装成taskRunner，并从线程池中抽取出一个空闲线程运行Task，这样，每个CoarseGrainedExecutorBackend能并行运行Task的数据就取决于分配给它的CPU的个数。
> * Worker：集群中可以运行Application代码的节点。在Standalone模式中指的是通过slave文件配置的worker节点，在Spark on Yarn模式中指的就是NodeManager节点。
> * Task：在Executor进程中执行任务的工作单元，多个Task组成一个Stage
> * Job：包含多个Task组成的并行计算，是由Action行为触发的
> * Stage：每个Job会被拆分很多组Task，作为一个TaskSet，其名称为Stage
> * DAGScheduler：根据Job构建基于Stage的DAG，并提交Stage给TaskScheduler，其划分Stage的依据是RDD之间的依赖关系
> * TaskScheduler：将TaskSet提交给Worker（集群）运行，每个Executor运行什么Task就是在此处分配的。

### Spark部署模式

> 1. **Local**:运行在一台机器上，通常是练手或者测试环境。
> 2. **Standalone**:构建一个基于Mster+Slaves的资源调度集群，Spark任务提交给Master运行。是Spark自身的一个调度系统。
> 3. **Yarn**: Spark客户端直接连接Yarn，不需要额外构建Spark集群。有yarn-client和yarn-cluster两种模式，主要区别在于：Driver程序的运行节点。
> 4. **Mesos**：国内大环境比较少用。

### Spark提交作业参数

> Spark任务是采用的Shell脚本进行提交
>
> - executor-cores —— 每个executor使用的内核数，默认为1，官方建议2-5个
> - num-executors —— 启动executors的数量，默认为2
> - executor-memory —— executor内存大小，默认1G
> - driver-cores —— driver使用内核数，默认为1
> - driver-memory —— driver内存大小，默认512M

### Spark提交作业流程

>Spark的任务提交方式实际上有两种，分别是YarnClient模式和YarnCluster模式。

#### YarnClient

> 在YARN Client模式下，Driver在任务提交的本地机器上运行，Driver启动后会和**ResourceManager**通讯申请启动**ApplicationMaster**，随后ResourceManager分配**container**，在合适的**NodeManager**上启动ApplicationMaster，此时的ApplicationMaster的功能相当于一个ExecutorLaucher，**只负责**向ResourceManager申请Executor内存。
>
>  
>
> ResourceManager接到ApplicationMaster的资源申请后会分配container，然后ApplicationMaster在资源分配指定的NodeManager上启动Executor进程，Executor进程启动后会向Driver反向注册，Executor全部注册完成后Driver开始执行main函数，之后执行到Action算子时，触发一个job，并根据宽依赖开始划分stage，每个stage生成对应的taskSet，之后将task分发到各个Executor上执行。
>
> ![image-20220318102708067](/img/image-20220318102708067.png)

#### YarnCluster

> 在YARN Cluster模式下，任务提交后会和**ResourceManager**通讯申请启动**ApplicationMaster**，随后ResourceManager分配**container**，在合适的**NodeManager**上启动ApplicationMaster，此时的ApplicationMaster就是**Driver**。
>
>  
>
> Driver启动后向ResourceManager申请Executor内存，ResourceManager接到ApplicationMaster的资源申请后会分配container，然后在合适的NodeManager上启动Executor进程，Executor进程启动后会向Driver反向注册，Executor全部注册完成后Driver开始执行main函数，之后执行到Action算子时，触发一个job，并根据宽依赖开始划分stage，每个stage生成对应的taskSet，之后将task分发到各个Executor上执行。
>
> ![image-20220318102856431](/img/image-20220318102856431.png)

#### yarn-client 和 yarn-cluster 的异同 

> 1. 从广义上讲，yarn-cluster 适用于生产环境。而 yarn-client 适用于交互和调试，也就是希望快速地看到 application 的输出。 
> 2. 从深层次的含义讲，yarn-cluster 和 yarn-client 模式的区别其实就是 Application Master 进程的区别，yarn-cluster 模式下，driver 运行在 AM(Application Master)中，它负责向 YARN 申请资源，并监督作业的运行状况。当用户提交了作业之后，就可以关掉 Client，作业会继续在 YARN 上运行。然而 yarn-cluster 模式不适合运行交互类型的作业。而 yarn-client 模式下，Application Master 仅仅向 YARN 请求 executor，Client 会和请求的 container 通信来调度他们工作，也就是说 Client 不能离开。

#### Spark on Yarn 优势

> * Spark 支持资源动态共享，运行于 Yarn 的框架都共享一个集中配置好的资源池 
> * 可以很方便的利用 Yarn 的资源调度特性来做分类·，隔离以及优先级控制负载，拥有更灵活的调度策略
> * Yarn 可以自由地选择 executor 数量 
> * Yarn 是唯一支持 Spark 安全的集群管理器（Mesos???），使用 Yarn，Spark 可以运行于 Kerberized Hadoop 之上，在它们进程之间进行安全认证

### Spark的任务执行流程

> 1. 构建Spark Application的运行环境（启动SparkContext），**SparkContext**向资源管理器（可以是Standalone、Mesos或YARN）注册并申请运行Executor资源；
>
> 2. 资源管理器分配**Executor**资源并启动**StandaloneExecutorBackend**，Executor运行情况将随着心跳发送到资源管理器上；
>
> 3. SparkContext构建成**DAG**图，将DAG图分解成Stage，并把Taskset发送给Task Scheduler。Executor向SparkContext申请Task
>
> 4. **Task Scheduler**将Task发放给Executor运行同时**SparkContext**将应用程序代码发放给Executor。
>
> 5. Task在**Executor**上运行，运行完毕释放所有资源。
>
> ![img](/img/1411500-20190519213716709-233373174.png)

### spark-submit的时候如何引入外部jar包

> * 在通过spark-submit提交任务时，可以通过添加**配置参数**来指定
> * --**driver-class-path** 外部jar包
> * --**jars** 外部jar包

### Spark 如何防止内存溢出

#### driver端的内存溢出

> * 可以增大driver的内存参数：spark.driver.memory (default 1g)
> * 这个参数用来设置Driver的内存。在Spark程序中，SparkContext，DAGScheduler都是运行在Driver端的。对应rdd的Stage切分也是在Driver端运行，如果用户自己写的程序有过多的步骤，切分出过多的Stage，这部分信息消耗的是Driver的内存，这个时候就需要调大Driver的内存。

####  map过程产生大量对象导致内存溢出

> * 这种溢出的原因是在单个map中产生了大量的对象导致的，例如：rdd.map(x=>for(i <- 1 to 10000) yield i.toString)，这个操作在rdd中，每个对象都产生了10000个对象，这肯定很容易产生内存溢出的问题。针对这种问题，在不增加内存的情况下，可以通过减少每个Task的大小，以便达到每个Task即使产生大量的对象Executor的内存也能够装得下。具体做法可以在会产生大量对象的map操作之前调用repartition方法，分区成更小的块传入map。例如：rdd.repartition(10000).map(x=>for(i <- 1 to 10000) yield i.toString)。
>
> 面对这种问题注意，不能使用rdd.coalesce方法，这个方法只能减少分区，不能增加分区，不会有shuffle的过程。

#### 数据不平衡导致内存溢出

> 数据不平衡除了有可能导致内存溢出外，也有可能导致性能的问题，解决方法和上面说的类似，就是调用repartition重新分区。这里就不再累赘了。

####  shuffle后内存溢出

> shuffle内存溢出的情况可以说都是shuffle后，单个文件过大导致的。在Spark中，join，reduceByKey这一类型的过程，都会有shuffle的过程，在shuffle的使用，需要传入一个partitioner，大部分Spark中的shuffle操作，**默认**的partitioner都是**HashPatitioner**，默认值是父RDD中最大的分区数,这个参数通过spark.default.parallelism控制(在spark-sql中用spark.sql.shuffle.partitions) ， spark.default.parallelism参数只对HashPartitioner有效，所以如果是别的Partitioner或者自己实现的Partitioner就不能使用spark.default.parallelism这个参数来控制shuffle的并发量了。如果是别的partitioner导致的shuffle内存溢出，就需要从partitioner的代码**增加partitions的数量**。

#### standalone模式下资源分配不均匀导致内存溢出

> 在standalone的模式下如果配置了--total-executor-cores 和 --executor-memory 这两个参数，但是没有配置--executor-cores这个参数的话，就有可能导致，每个Executor的memory是一样的，但是cores的数量不同，那么在cores数量多的Executor中，由于能够同时执行多个Task，就容易导致内存溢出的情况。这种情况的解决方法就是同时配置**--executor-cores**或者**spark.executor.cores**参数，确保Executor资源分配均匀。

#### 使用rdd.persist(StorageLevel.MEMORY_AND_DISK_SER)代替rdd.cache()

>rdd.cache()和rdd.persist(Storage.MEMORY_ONLY)是等价的，在内存不足的时候rdd.cache()的数据会丢失，再次使用的时候会重算，而rdd.persist(StorageLevel.MEMORY_AND_DISK_SER)在内存不足的时候会存储在磁盘，避免重算，只是消耗点IO时间。

### Spark中的RDD (Resilient Distributed Datasets)

> RDD是**弹性分布式数据集**，是Spark中最基本的数据抽象，代表一个不可变、可分区、里面的元素可并行计算的集合。
>
>  
>
> 作用：提供了一个抽象的数据模型，将具体的应用逻辑表达为一系列转换操作(函数)。另外不同RDD之间的转换操作之间还可以形成依赖关系，进而实现管道化，从而避免了中间结果的存储，大大降低了数据复制、磁盘IO和序列化开销，并且还提供了更多的API(map/reduec/filter/groupBy...)。
>
>  
>
> RDD在Lineage依赖方面分为两种Narrow Dependencies与Wide Dependencies，用来解决数据容错时的高效性以及划分任务时候起到重要作用。
>
> 
>
> **一个 RDD 代表一个可以被分区的只读数据集。RDD 内部可以有许多分区(partitions)，每个分区又拥有大量的记录(records)**。

#### RDD五大特性

> 1. dependencies: 建立 RDD 的依赖关系，主要 RDD 之间是宽窄依赖的关系，具有窄依赖关系的 RDD 可以在同一个 stage 中进行计算。 
>
> 2. partition: 一个 RDD 会有若干个分区，分区的大小决定了对这个 RDD 计算的粒度，每个 RDD 的分区的计算都在一个单独的任务中进行。 
>
> 3. preferedlocations: 按照“移动数据不如移动计算”原则，在 Spark 进行任务调度的时候，优先将任务分配到数据块存储的位置。
>
> 4. compute: Spark 中的计算都是以分区为基本单位的，compute 函数只是对迭代器进行复合，并不保存单次计算的结果。 
>
> 5. partitioner: 只存在于（K,V）类型的 RDD 中，非（K,V）类型的 partitioner 的值就是 None。

#### RDD算子

> RDD 的**算子**主要分成2类，**action 和 transformation**。这里的算子概念，可以理解成就是**对数据集的变换**。
>
> 
>
> action 会触发真正的作业提交，而 transformation 算子是不会立即触发作业提交的。每一个 transformation 方法返回一个新的 RDD。只是某些 transformation 比较复杂，会包含多个子 transformation，因而会生成多个 RDD。这就是实际 RDD 个数比我们想象的多一些 的原因。通常是，当遇到 action 算子时会触发一个job的提交，然后反推回去看前面的 transformation 算子，进而形成一张有向无环图。

### Spark为什么快，Spark SQL 一定比 Hive 快吗

> **Spark SQL** **比 Hadoop Hive** **快**，是有一定条件的，而且不是 Spark SQL 的引擎比 Hive 的引擎快，相反，Hive的**HQL引擎**还比Spark SQL的引擎**更快**。关键还是在于Spark本身快。
>
>  
>
> **消除了冗余的 HDFS** **读写**: Hadoop 每次 shuffle 操作后，必须写到磁盘，而 Spark 在 shuffle 后不一定落盘，可以 cache 到内存中，以便迭代时使用。如果操作复杂，很多的 shufle 操作，那么 Hadoop 的读写 IO 时间会大大增加，也是 Hive 更慢的主要原因了。
>
> 消除了冗余的 MapReduce 阶段: Hadoop 的 shuffle 操作一定连着完整的 MapReduce 操作，冗余繁琐。而 Spark 基于 RDD 提供了丰富的算子操作，且 reduce 操作产生 shuffle 数据，可以缓存在内存中。
>
> **JVM** **的优化**: Hadoop 每次 MapReduce 操作，启动一个 Task 便会启动一次 JVM，基于进程的操作。而 Spark 每次 MapReduce 操作是基于线程的，只在启动 Executor 是启动一次 JVM，内存的 Task 操作是在线程复用的。每次启动 JVM 的时间可能就需要几秒甚至十几秒，那么当 Task 多了，这个时间 Hadoop 不知道比 Spark 慢了多少。
>
> 记住一种反例 考虑一种极端查询:
>
> Select month_id, sum(sales) from T group by month_id;
>
> 这个查询只有一次 shuffle 操作，此时，也许 Hive HQL 的运行时间也许比 Spark 还快，反正 shuffle 完了都会落一次盘，或者都不落盘。
>
>  
>
> 结论 Spark 快不是绝对的，但是绝大多数，Spark 都比 Hadoop 计算要快。这主要得益于**其对** **mapreduce** **操作的优化以及对 JVM** **使用的优化**。

### Spark调优

#### 资源参数调优

> * num-executors：设置Spark作业总共要用多少个Executor进程来执行
> * executor-memory：设置每个Executor进程的内存
> * executor-cores：设置每个Executor进程的CPU core数量
> * driver-memory：设置Driver进程的内存
> * spark.default.parallelism：设置每个stage的默认task数量

#### 开发调优

> * 避免创建重复的RDD
> * 尽可能复用同一个RDD
> * 对多次使用的RDD进行持久化
> * 尽量避免使用shuffle类算子
> * 使用map-side预聚合的shuffle操作
> * 使用高性能的算子

### Spark Shuffle

> 混洗，官方定义是：一种让数据重新分布以使得某些数据被放在同一分区里的一种机制。Shuffle的过程中，存在着大量的网络消耗传输数据，会在磁盘上产生大量的中间文件。

#### 调优参数

> **spark.shuffle.file.buffer**
> **参数说明**：**该参数用于设置shuffle write task的BufferedOutputStream的buffer缓冲大小**（默认是32K）。将数据写到磁盘文件之前，会先写入buffer缓冲中，待缓冲写满之后，才会溢写到磁盘。
> **调优建议**：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如64k），从而减少shuffle write过程中溢写磁盘文件的次数，也就可以减少磁盘IO次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。
>  
> **spark.reducer.maxSizeInFlight**：
> **参数说明**：**该参数用于设置shuffle read task的buffer缓冲大小**，而这个buffer缓冲决定了每次能够拉取多少数据。
> **调优建议**：如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如96m），从而减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能。在实践中发现，合理调节该参数，性能会有1%~5%的提升。
>  
> **spark.shuffle.io.maxRetries & spark.shuffle.io.retryWait**：
> **spark.shuffle.io.retryWait**：huffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的。**该参数就代表了可以重试的最大次数**。（默认是3次）
> **spark.shuffle.io.retryWait**：**该参数代表了每次重试拉取数据的等待间隔**。（默认为5s）
> **调优建议**：一般的调优都是将重试次数调高，不调整时间间隔。
>  
> **spark.shuffle.memoryFraction**：
> **参数说明**：该参数代表了Executor内存中，分配给shuffle read task进行聚合操作的内存比例。
>  
> **spark.shuffle.manager**
> **参数说明**：**该参数用于设置shufflemanager的类型**（默认为sort）。Spark1.5x以后有三个可选项：
>
> ```lua
> Hash：spark1.x版本的默认值，HashShuffleManager
> Sort：spark2.x版本的默认值，普通机制，当shuffle read task 的数量小于等于spark.shuffle.sort.bypassMergeThreshold参数，自动开启bypass 机制
> tungsten-sort：
> ```
>
> **spark.shuffle.sort.bypassMergeThreshold**
> **参数说明**：当ShuffleManager为SortShuffleManager时，如果shuffle read task的数量小于这个阈值（默认是200），则shuffle write过程中不会进行排序操作。
> **调优建议**：当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些
>  
> **spark.shuffle.consolidateFiles**：
> **参数说明**：如果使用HashShuffleManager，该参数有效。如果设置为true，那么就会开启consolidate机制，也就是开启优化后的HashShuffleManager。
> **调优建议**：如果的确不需要SortShuffleManager的排序机制，那么除了使用bypass机制，还可以尝试将spark.shffle.manager参数手动指定为hash，使用HashShuffleManager，同时开启consolidate机制。在实践中尝试过，发现**其性能比开启了bypass机制的SortShuffleManager要高出10%~30%**。
