## 概述
开始之前先看看其基本属性，HDFS（Hadoop Distributed File System）是GFS的开源实现。

#### 特点如下：

*  能够运行在廉价机器上，硬件出错常态，需要具备高容错性
* 流式数据访问，而不是随机读写
* 面向大规模数据集，能够进行批处理、能够横向扩展
* 简单一致性模型，假定文件是一次写入、多次读取
#### 缺点：

* 不支持低延迟数据访问
* 不适合大量小文件存储（因为每条元数据占用空间是一定的）
* 不支持并发写入，一个文件只能有一个写入者
* 不支持文件随机修改，仅支持追加写入
### HDFS中的block、packet、chunk

很多博文介绍HDFS读写流程上来就直接从文件分块开始，其实，要把读写过程细节搞明白前，你必须知道block、packet与chunk。下面分别讲述。

1. block
这个大家应该知道，文件上传前需要分块，这个块就是block，一般为128MB，当然你可以去改，不顾不推荐。因为
    ##### 块太小：寻址时间占比过高。块太大：Map任务数太少，作业执行速度变慢。它是最大的一个单位。

2. packet
packet是第二大的单位，它是client端向DataNode，或DataNode的PipLine之间传数据的基本单位，默认64KB。

3. chunk
chunk是最小的单位，它是client向DataNode，或DataNode的PipLine之间进行数据校验的基本单位，默认512Byte，因为用作校验，故每个chunk需要带有4Byte的校验位。所以实际每个chunk写入packet的大小为516Byte。由此可见真实数据与校验值数据的比值约为128 : 1。（即64*1024 / 512）

例如，在client端向DataNode传数据的时候，HDFSOutputStream会有一个chunk buff，写满一个chunk后，会计算校验和并写入当前的chunk。之后再把带有校验和的chunk写入packet，当一个packet写满后，packet会进入dataQueue队列，其他的DataNode就是从这个dataQueue获取client端上传的数据并存储的。同时一个DataNode成功存储一个packet后之后会返回一个ack packet，放入ack Queue中。

### HDFS写流程

![Image text](https://github.com/1367379258/BigDataEd/blob/master/hadoop/photo/HDFS%E5%86%99%E6%B5%81%E7%A8%8B.jpg)

写详细步骤：
1. client发起文件上传请求,通过RPC与NameNode建立连接,NameNode检查目标文件是否已经存在,父目录是否存在,并检查用户是否有相应的权限,若检查通过,  
会为该文件创建一个新的记录,否则的话文件创建失败,客户端得到异常信息,  
2. client开始分块 ，通过请求NameNode,第一个block应该传输到哪些DataNode服务器上;  
3. NameNode根据配置文件中指定的备份(replica)数量及机架感知原理进行文件分配,返回可用的DataNode的地址 以三台DataNode为例:A B C  
> 注: Hadoop在设计时考虑到数据的安全与高效,数据文件默认在HDFS上存放三份,存储策略为:第一个备份放在客户端相同的datanode上(若客户端在集群外运行,  
	就随机选取一个datanode来存放第一个replica),第二个replica放在与第一个replica不同机架的一个随机datanode上,第三个replica放在与第二个repli  
	ca相同机架的随机datanode上,如果replica数大于三,则随后的replica在集群中随机存放,Hadoop会尽量避免过多的replica存放在同一个机架上.选取repl  
	ica存放在同一个机架上.(Hadoop 1.x以后允许replica是可插拔的,意思是说可以定制自己需要的replica分配策略)  

4. client请求3台的DataNode的一台A上传数据,(本质是一个RPC调用,建立pipeline),A收到请求会继续调用B,然后B调用C,将整个pipeline建立完成后,逐  
级返回client;  
5. client开始往A上传第一个block(先从磁盘读取数据放到一个本地内存缓存),以packet为单位(默认 64K),A收到一个packet就会传给B,B传递给C;A每传一  
个packet会放入一个应答队列等待应答.   这个wait queue在client等待。
>	
	注: 如果某个datanode在写数据的时候宕掉了下面这些对用户透明的步骤会被执行:  
	1) 管道线关闭,所有确认队列上的数据会被挪到数据队列的首部重新发送,这样也就确保管道线中宕掉的datanode下流的datanode不会因为宕掉的datanode而丢  
	失数据包  
	2) 在还在正常运行datanode上的当前block上做一个标志,这样当宕掉的datanode重新启动以后namenode就会知道该datanode上哪个block是刚才宕机残留下  
	的局部损坏block,从而把他删除掉  
	3) 已经宕掉的datanode从管道线中被移除,未写完的block的其他数据继续呗写入到其他两个还在正常运行的datanode中,namenode知道这个block还处在und  
	er-replicated状态(即备份数不足的状态)下,然后它会安排一个新的replica从而达到要求的备份数,后续的block写入方法同前面正常时候一样  
	4) 有可能管道线中的多个datanode宕掉(一般这种情况很少),但只要dfs.relication.min(默认值为1)个replica被创建,我么就认为该创建成功了,剩余的re  
	lica会在以后异步创建以达到指定的replica数.  
6. 数据被分割成一个个packet数据包在pipeline上一次传输,在pipeline反方向上,逐个发送ack(命令正确应答),最终由pipeline中第一个DataNode节点A  
	将pipeline ack发送给client  ,这些 ack 放在ack queue中
7. 当一个block传输完成后,client再次发送请求NameNode上传第二个block到服务器  
7. 发送完成信号给NameNode。
> （注：发送完成信号的时机取决于集群是强一致性还是最终一致性，强一致性则需要所有DataNode写完后才向NameNode汇报。最终一致性则其中任意一个DataNode写完后就能单独向NameNode汇报，HDFS一般情况下都是强调强一致性）


### HDFS读流程

![Image text](https://github.com/1367379258/BigDataEd/blob/master/hadoop/photo/HDFS%E8%AF%BB%E6%B5%81%E7%A8%8B.jpg)

读相对于写，简单一些
读详细步骤：

1. 跟namenode通信查询元数据，找到文件块所在的datanode服务器，HDFS客户端首先调用DistributedFileSystem.open方法打开HDFS文件，底层会调用ClientProtocal.open方法，返回一个用于读取的HdfsDataInputStream对象。

2. 从NameNode获取DataNode地址：在构造DFSInputStream的时候，对调用ClientPortocal.getBlockLocations方法向NameNode获取该文件起始位置数据块信息。NameNode返回的数据块的存储位置是按照与客户端距离远近排序的。所以DFSInputStream可以选择一个最优的DataNode节点,然后与这个节点建立数据连接读取数据块。

3. 连接到DataNode读取数据块：
HDFS客户端通过调用DFSInputSttream从最优的DataNode读取数据块，数据会以数据包packet形式从DataNode以流式接口传送到客户端，当达到一个数据块末尾的时候,DFSInputStream就会再次调用ClientProtocal.getBlockL
octions获取下一个数据块的位置信息，并建立和这个新的数据块的最优节点之间的连接，然后HDFS继续读取数据块。

>
	(客户端读取数据块的时候，很有可能这个数据块的DataNode出现异常，也就是无法读取数据。这时候DFSInputStream会切换到另一个保存了这个数据块副本的DataNode，然后读取数据。另外，数据块的应答不仅包含了数据块还包含了校验值，HDFS客户端收到数据应答包的时候，会对数据进行校验，如果校验错误，也就是DataNode这个数据块副本出现了损坏，HDFS
	客户端会通过ClientProtocal.reportBadBlocks向NameNode汇报这个损坏的数据块副本，同时DFSInputStream会尝试从其他DataNode读取这个数据块)

4.客户端关闭输入流

### 读写过程，数据完整性如何保持？
通过校验和。因为每个chunk中都有一个校验位，一个个chunk构成packet，一个个packet最终形成block，故可在block上求校验和。

> HDFS 的client端即实现了对 HDFS 文件内容的校验和 (checksum) 检查。当客户端创建一个新的HDFS文件时候，分块后会计算这个文件每个数据块的校验和，此校验和会以一个隐藏文件形式保存在同一个 HDFS 命名空间下。当client端从HDFS中读取文件内容后，它会检查分块时候计算出的校验和（隐藏文件里）和读取到的文件块中校验和是否匹配，如果不匹配，客户端可以选择从其他 Datanode 获取该数据块的副本。

> 
	HDFS中文件块目录结构具体格式如下：

	${dfs.datanode.data.dir}/
	├── current
	│ ├── BP-526805057-127.0.0.1-1411980876842
	│ │ └── current
	│ │ ├── VERSION
	│ │ ├── finalized
	│ │ │ ├── blk_1073741825
	│ │ │ ├── blk_1073741825_1001.meta
	│ │ │ ├── blk_1073741826
	│ │ │ └── blk_1073741826_1002.meta
	│ │ └── rbw
	│ └── VERSION
	└── in_use.lock

> in_use.lock表示DataNode正在对文件夹进行操作

> rbw是“replica being written”的意思，该目录用于存储用户当前正在写入的数据。
	Block元数据文件（*.meta）由一个包含版本、类型信息的头文件和一系列校验值组成。校验和也正是存在其中。

