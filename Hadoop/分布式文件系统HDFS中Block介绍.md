Block是一块磁盘当中最小的单位，HDFS中的Block是一个很大的单元。在HDFS中的文件将会按块大小进行分解，并作为独立的单元进行存储。

### Block概念
磁盘有一个Block size的概念，它是磁盘读/写数据的最小单位。构建在这样的磁盘上的文件系统也是通过块来管理数据的，文件系统的块通常是磁盘块的整数倍。文件系统的块一般为几千字节(byte)，磁盘块一般为512字节(byte)。
HDFS也有Block的概念，但它的块是一个很大的单元，默认是64MB。像硬盘中的文件系统一样，在HDFS中的文件将会按块大小进行分解，并作为独立的单元进行存储。但和硬盘中的文件系统不一样的是，存储在块中的一个比块小的文件并不会占据一个块大小盘物理空间(HDFS中一个块只存储一个文件的内容)。

### 那为什么HDFS中的块如此之大呢?
在HDFS学习(一) – HDFS设计中，我们曾说过，对HDFS来说，读取整个数据的时间延迟要比读取到第一条记录的数据延迟更重要，就体现在这里。HDFS的Block设计的如此之大，也就是为了最小化寻道时间。把一个数据块设计的足够大，就能够使得数据传输的时间显著地大于寻找到Block所在时间。这样，传输一个由多个Block组成的文件的时间就取决于磁盘的传输速率。
举一个简单的例子，假设寻道时间大约为10ms，传输速度为100MB/s。为了使得寻道时间仅为传输时间的1%，我们就需要设置块的大小为100MB。HDFS默认的Block size是64MB，但是更多的企业里边，已经设置成128M，而且这个参数将随着新一代硬盘速度的增长而增长。
而Block Size的值也不宜设置过大，通常，Mapreduce中的Map任务一次只处理一个Block中的数据，如果启动太少的Task(少于集群中的节点的数量)，作业的速度就会比较慢。

### 对HDFS进行块抽象有哪些好处呢？
一、一个显而易见的好处是：一个文件的大小，可以大于网络中任意一个硬盘的大小。
文件的块并不需要存储在同一个硬盘上，一个文件的快可以分布在集群中任意一个硬盘上。事实上，虽然实际中并没有，整个集群可以只存储一个文件，该文件的块占满整个集群的硬盘空间。
二、使用抽象块而非整个文件作为存储单元，大大简化了系统的设计。
简化设计，对于故障种类繁多的分布式系统来说尤为重要。以块为单位，一方面简化存储管理，因为块大小是固定的，所以一个硬盘放多少个块是非常容易计算的;另一方面，也消除了元数据的顾虑，因为Block仅仅是存储的一块数据，其文件的元数据，例如权限等就不需要跟数据块一起存储，可以交由另外的其他系统来处理。
三、块更适合于数据备份，进而提供数据容错能力和系统可用性。
为了防止数据块损坏或者磁盘或者机器故障，每一个block都可以被分到少数几台独立的机器上(默认3台)。这样，如果一个block不能用了，就从其他的一处地方，复制过来一份。详细的关于HDFS数据完整性的问题，将在《Hadoop I/O学习(一) – 数据完整性》中详细介绍。