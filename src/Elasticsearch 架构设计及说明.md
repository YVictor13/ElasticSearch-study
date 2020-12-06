# Elasticsearch 架构设计及说明

**Elasticsearch 架构层：**

![image-20201206142828286](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201206142828286.png)

**Elasticsearch 五层架构设计图：**

![image-20201206144157817](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201206144157817.png)

**说明：**

- **Elasticsearch**是基于**Lucene**架构实现的，所以其核心层为**Lucene**
- **Elasticsearch** 目前支持**HTTP**、**Thrift**、**Memcache**三种协议，默认是**HTTP** 
- **JMX**指在**Elasticsearch**中对 **Java** 的管理架构，用来管理**Elasticsearch**应用



## 一、节点自动发现机制

```
* 在进行Elasticsearch 分布式安装过程中，我们通过在elasticsearch.yml文件中配置 ‘cluster.name’就能将不同的节点连接到同一个
集群中，正是使用了Elasticsearch的节点发现机制。Elasticsearch内置有默认发现模块，同时也支持插件方式的发现机制。
* 发现机制用于发现集群中的节点和选举主节点（master节点）
```

**Elasticsearch内嵌自动发现机制**：

- **Azure discovery 插件方式**：多播模式
- **EC2 discovery 插件方式**：多播模式
- **Google Compute Engine discovery 插件方式**：多播模式
- **Zen Discovery **：默认实现方式，支持多播模式和单播模式。提供了单播模式和基于文件的发现，并且可以扩展为通过插件支持其他形式的发现机制。

**单播模式和多播模式配置参数**：

- **discovery.zen.ping.multicast.enabled**：表示关闭多播模式的自动发现机制，主要为了防止其他机器上的节点自动接入。
- **discovery.zen.fd.ping_timeout** 与 **discovery.zen.ping.timeout**：表示设置了节点和节点之间连接**ping**命令执行的超时时长。
- **discovery.zen.minimum_master_nodes**：表示集群中选举主节点时至少需要有多少个节点参与
- **discovery.zen.ping.unicast.hosts**：表示在单播模式下，节点应该自动发现哪些节点列表
- **action_create_index**：**false**表示关闭自动创建索引

**如：**

```
discovery.zen.ping.multicast.enabled:true
discovery.zen.fd.ping_timeout:100s
discovery.zen.ping.timeou:100s
discovery.zen.minimum_master_nodes:2
discovery.zen.ping.unicast.hosts:['192.168.x1.y1:9300','192.168.x2.y2:9300']
action_create_index:false
```



### 1、单播模式

```
* Elasticsearch支持同一台主机启动多个节点，所以只有在同一台机器上运行的节点才会自动组成集群。
* 单播模式下，需要配置Elasticsearch，填写集群中的IP地址列表。如下：
discovery.zen.ping.unicast.hosts:['192.168.x1.y1:9300','192.168.x2.y2:9300']
```

**单播模式下的配置信息**：

```
discovery.zen.ping.multicast.enabled:false
discovery.zen.fd.ping_timeout:100s
discovery.zen.ping.timeou:100s
discovery.zen.minimum_master_nodes:2
discovery.zen.ping.unicast.hosts:['192.168.x1.y1:9300','192.168.x2.y2:9300']
```

**集群构建及主节点选举过程：**

1. 节点启动后执行[ping](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/16.x/api-ping-7-3.html)命令（RPC命令），如果```discovery.zen.ping.unicast.hosts ```中有设置，则ping设置hosts中，否则尝试ping localhost的几个端口。最后返回包含节点的基本信息以及该节点认为的主节点。
2. 从各个节点认为的master中，按照ID的字典排序选取第一个为主节点。如果各个节点没有认为的master，则从各个节点中按照ID字典排序选取第一个作为主节点。

**注：**

- 在选举主节点时，如果节点数达不到最小值得限制（`` discovery.zen.minimum_master_nodes:num``），则循环集群构建与主节点选举过程，直到节点数超过最小限制值，才可以开始选举主节点。
- 如果只有一个本地节点，则主节点就是它自己。
- 如果当前节点是主节点了，则需要等到节点数达到限制值，再提供服务
- 如果当前节点不是主节点，则尝试加入主节点所在的集群



### 2、多播模式

```java
* 多播模式下，只需在每个节点配置好集群名称和节点名称即可。互相通信的节点会更加Elasticsearch自定义的服务发现协议，按照多播的方式
寻找网络上配置在同样集群内的节点。
* 【注】虽然Elasticsearch支持多播和单播模式两种自动节点发现机制，但是多播模式安全性不高，已经不被多大数操作系统支持。目前
Elasticsearch默认发现机制为单播模式,以防止节点无意中加入集群。所以是配置中需要将多播模式关闭，如下：
discovery.zen.ping.multicast.enabled: false 
```



## 二、节点类型

```
节点角色在配置文件（/config）elasticsearch.yml文件中设置即可，如下：
# 是否为候选主节点
node.master:true
# 是否为数据节点
node.data:true
```

![image-20201206161711131](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201206161711131.png)



## 三、分片和路由

```
由于Elasticsearch中，在一个多分片的索引中写入数据时，需要通过路由来确定具体陷入哪一个分片中，所以在创建索引时需要指定分片数量，
配置副本，并且分片的数量一旦确定就不能修改。
```

### 1、分片

通过配置创建索引时的Setting来配置分片和副本数量，如下：

```
index.number_of_shards:5
index.number_of_replicas:1
```

**补充说明：**

- 对文档的新建、索引和删除请求等写操作，必须在主分片上面完成之后才能被复制到副本分片。
- Elasticsearch 使用乐观锁来控制加快写入速度的并发写入引起的数据冲突问题，通过为每个文档设置一个version(版本号)，当文档被修改时版本号递增来实现。
- 当向Elasticsearch写入数据时，Elasticsearch根据文档标识符ID将文档分配到多个分片上，当查询数据时，Elasticsearch会查询所有的分片并汇总结果。
- 对于分片，用户并不知道数据存在哪个分片上。



### 2、路由

```
为了避免查询时部分分片查询失败影响结果的准确性，Elasticsearch引入了路由功能。
当数据写入时，通过路由将数据写入指定分片；
当查询数据时，通过相同的路由指明在哪个分片将数据查出来
```

**索引数据分片算法：**

```
shard_num = hash(_routing) % num_primary_shards  #_routing默认为id字段或者parent字段
```

**补充说明：**

- 通过 **Hash** 分片来保证在每个分片上的数据量能均匀分布，避免各个分片的存储负载不均衡
- 在做数据检索时，Elasticsearch默认会搜索所有分片上的数据，最后再主节点上汇总各个分片数据并排序处理，返回最终的结果数据。



## 四、Elasticsearch 数据写入过程

**数据存储路径配置：**（/config）elasticsearch.yml文件中

```
path.data:/path/to/data # 索引数据
path.logs:/path/to/logs # 日志数据

【注】不建议使用默认值，防止升级Elasticsearch而导致数据部分甚至全部丢失
```

### 1、分段存储

```
* 在索引中，索引文件被拆分为多个子文件，其中每个子文件就叫做段，且每个段都是一个倒排索引的小单元
* 索引数据在磁盘中是以分段形式存储的
* 段具有不可变性，一旦索引的数据被写入硬盘，就不能在修改
```

我们可以思考一下，为什么Elasticsearch中数据的存储要引入段？

```
* 如果熟悉CurrentHashMap原理的，可以想到其中的缘由，引入段就是为了减少锁的使用，提高并发。
* 假设我们将全部文档集合构建在一个很大的倒排索引文件中，且数据还在不断增加，那么当我们进行修改时，就需要全量更新当前的倒排索引文件
这样就使得数据更新变得时效性很差，且耗费大量资源。
* 引入段的存储模式，避免了在读写操作时使用锁，从而大大提升Elasticsearch的读写性能。
```

当段被写入**磁盘**后会生成一个提交点，并生成一个用来记录所有段信息的文件，则对于该文件段只有读的权限，永远失去写的权限。同时该部分数据可以被 **Elasticsearch** 用户检索到。而当段还在**内存**中时，此时分段只拥有写的权限，数据还能不断写入，但不具备读数据的权限，且无法被 **Elasticsearch**  用户检索到。

如果段一旦提交不能再写，那么我们如何进行 ’**改**‘ （新增、更新和删除）的操作呢？

```
> 在面对段的不可修改特性，Elasticsearch采用不将文档从旧段中移除，而是新增一个.del文件，记录被 '改' 文档的段信息。
> 当用户检索时，文档依然可以被查询到，但他会在最终结果被返回前通过.del文件将其从结果集中移除。
> 如当更新数据时，会先创建一个段，然后将更新好的数据写入新段中，生成提交点，再在.del文件中标记旧段，从而达到更新的效果。
```

**段的优缺点：**

- **优点**：不需要锁，大大提升了 **Elasticsearch** 的读写性能
- **缺点1**：存储空间占用大，如当删除旧数据时，旧数据不会被马上删除，而是在.del文件中标记为删除，只有等到段更新时才被移除。这就导致了存储空间的浪费，倘若频繁更新数据，则每次更新都是新增新的数据到新分段，并标记旧的分段中的数据，存储空间的浪费会更多。
- **缺点2**：在检索数据时，检索得到的数据集中包含所有的结果集，因此主节点需要排除被标记删除的旧数据，进而增加了查询负担。



### 2、延迟写策略

```
Elasticsearch中，索引写入磁盘是异步写入的。为了提升写的性能，延迟写策略采用了 '延迟写策略' 来解决新增一条数据就添加一个段
到磁盘上的问题。
```

**延迟写策略执行过程：**

![image-20201206194151188](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201206194151188.png)

**补充说明**：

- **JVM**内存中的数据不以段形式存储，无法提供检索功能

- 当生成段后便可以提供检索功能，无需等到刷新到磁盘。

- 刷新除了自动刷新，开发人员可以手动触发刷新

- 可通过在创建索引时的**Setting**文件中配置 ```refresh_interval```的值，来调整索引的刷新频率。如

  ```
  refresh_interval = number 时间单位  # 设置值时需要注意带上时间单位，否则默认为ms（毫秒）
  refresh_interval = -1 # 表示关闭索引的自动刷新
  ```

**延迟写策略优缺点：**

- **优点**：减少数据往磁盘上写的次数，提升**Elasticsearch**的整体写入能力
- **缺点**：引入数据丢失风险，如机器断电等。

```
为了解决延迟写策略引入的数据丢失风险，**Elasticsearch**又引入了 事务日志（**Translog**）机制，用于记录所有还没有持久化到磁盘
的数据。
```

**添加事务日志机制后的数据写入索引流程：**

1. 新文档被索引之后，先被写入内存中。为了防止数据丢失，**Elasticsearch**会追加一份数据到事务日志中。
2. 新的文档持续在被写入内存时，同时也会记录到事务日志中。此时，新数据不能被检索和查询
3. 当达到默认的刷新时间或内存中的数据达到一定量后，触发一次**Refresh**刷新将内存中的数据以一个新段格式刷新到文件缓存系统中，并清空内存。（形成段，可提供检索和查询，且不可修改）
4. 随着新文档索引不断写入，当日志数据大小超过某个值(**512MB**)，或者超过一定时间（**30min**）时，触发一次**Flush**，此时文件缓存系统中的数据通过**Fsync**刷新到磁盘中，生成提交点。同时删除日志文件，并创建一个空的新日志文件。

![image-20201206195233716](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201206195233716.png)



### 3、段合并

```
* ElasticSearch自动刷新阶段，每秒都会创建一个新段。因此，极短时间内会产生大量段，消耗较大的资源，如文件句柄、内存和CPU。
* 检索阶段，由于搜索要求检查到每个段，然后合并查询结果，因此段越多，搜索速度越慢。
```

为了解决段增多的问题，**Elasticsearch**引入了段合并机制，定期将较小的段合并到较大的段中，而较大的段合并到更大的段中；

**说明：**

- 在段合并过程中，**Elasticsearch**会将旧的\已删除的文件从文件系统中清除，从而保证旧的\删除的文档不会被拷贝到新的大段中；
- 段合并机制是自动进行索引和搜索的，它们会选择一小部分大小相似的段，在后台将它们合并到更大的段中。
- 合并的段可以是未提交的，也可以是已提交的。
- 合并结束后，老的段会被删除，新段被**Refresh**到磁盘，同时写入一个包含新段且排除旧的和较小段的新提交点。
- 由于段合并需要很大的计算量，因此，**Elasticsearch**在默认情况下会对段合并流程进行资源限制。

[上一篇：Elasticsearch 核心概念](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/Elasticsearch%20%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.md)

[下一篇：Elasticsearch ](#)