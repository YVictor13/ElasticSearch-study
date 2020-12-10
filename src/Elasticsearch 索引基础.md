# Elasticsearch 索引基础

安装 [分布式的 Elasticsearch](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/ElasticSearch%20%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97.md) ，启动一个master 和两个 slave 进行操作，以及 启动 [Kibana](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/ElasticSearch%20%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97.md)。

## 一、创建索引

### 1.1、ElasticSearch Head 插件创建

1. 进入ElasticSearch Head 插件，点击索引分页选项卡

   ![image-20201209204233770](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209204233770.png)

2. 点击新建索引，输入索引名称（索引名称不可重复）和选择分片与副本数

   ![image-20201209204348444](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209204348444.png)

3. 创建完成，如下：

   ![image-20201209205301416](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209205301416.png)

**补充：**

- 创建完索引之后，在ElasticSearch Head插件中可以看到如下图。
- 其中 加粗框的表示主分片，细框的表示副本。所以 0、1、2、3、4表示索引(**index**)的分片，而左边的**Kibana_1**索引就只有一个主分片和一个副本。

![image-20201209204456362](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209204456362.png)

- 点击 框，可以看到如下图，根据其中的**primary**属性查看是否为主分片。

  1. 主分片

     ![image-20201209205006699](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209205006699.png)

  2. 副本

     ![image-20201209205054669](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209205054669.png)

### 1.2、请求方式创建索引

#### 1.2.1、Postman 请求方式创建索引

```
PUT方式  发送 http://localhost:9200/test
```

![image-20201209205659625](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209205659625.png)

创建成功：

![image-20201209205738537](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209205738537.png)

#### 1.2.1、Kibana 请求方式创建索引

启动Kibana后，浏览器输入```localhost:5601```，进入**Dev Tools**

![image-20201209205944910](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209205944910.png)

输入命令  【 PUT  索引名 】，点击命令后面的执行图标，即可。如下：

```
PUT test  # 默认分片数和副本数

PUT test  # 设置分片数和副本数
{
	"settings": {
		"number_of_shards": 3,
		"number_of_replicas": 2
	}
}
```

![image-20201209210141354](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209210141354.png)

创建成功：

![image-20201209210202910](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209210202910.png)

![image-20201209210223214](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209210223214.png)

但是请求方式创建的索引，Elasticsearch 默认为分片数和副本数都为1
![image-20201209210401758](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209210401758.png)

**补充：**

- 索引名不能重复，唯一的。重复创建索引错误：

  ![image-20201209210553037](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209210553037.png)

- 索引名称不能有大写字母。创建含有大写字母索引错误：

  ![image-20201209210649368](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209210649368.png)

- 创建索引时，需要注意分片数的分配，分片数创建完后便不可在修改，建议使用带有分片数和副本数配置的方式创建索引（副本可修改）

  ![image-20201209222334388](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209222334388.png)

  ![image-20201209222252076](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209222252076.png)



## 二、更新索引

在通过请求方式创建完索引之后，我们发现索引的分片数和副本数都为1，一般情况下，不能满足我们的需求。那么我们该怎么修改索引的分片数和副本数属性呢？

```
PUT 索引名称/_settings
{
  "number_of_replicas": number
}

如：
PUT test/_settings
{
   "number_of_replicas": 2
}
```

修改成功：

![image-20201209211816377](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209211816377.png)

![image-20201209211858390](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209211858390.png)



这时就有人问，那么分片呢？还记得我们讲解[ElasticSearch 的架构设计与说明](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/Elasticsearch%20%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E5%8F%8A%E8%AF%B4%E6%98%8E.md)中说的，分片与路由吗？它是通过一个**hash**来路由分片存储，所以修改分片后就导致路由乱套，就导致分片找不到。

但是我们可以在创建索引之前，修改默认的分片数。Elasticsearch 默认分片数量为100。可使用如下请求方式修改为10000

```
PUT /_cluster/settings
{
	"persistent":{
		"cluster":{
		"max_shards_per_node":10000
		}
	}
}
```

![image-20201209212538321](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209212538321.png)



## 三、修改索引读写权限

### 3.1、添加文档

```
PUT 索引名/_doc/编号
{
  "titile":"我和我的爱人"
}

如：
PUT index/_doc/001
{
  "titile":"我和我的爱人"
}
```

添加成功：

![image-20201209222828210](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209222828210.png)

![image-20201209223005684](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209223005684.png)

默认设置中，索引具备读写权限，但读写权限可关闭：

**关闭索引读权限：**

```
PUT 索引名称/_settings
{
	"blocks.read":true
}

如：
PUT index/_settings
{
	"blocks.read":true
}
```

![image-20201209223411800](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209223411800.png)

![image-20201209223441807](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209223441807.png)

**关闭索引写权限：**

```
PUT 索引名称/_settings
{
	"blocks.write":true
}

如：
PUT index/_settings
{
	"blocks.write":true
}
```

![image-20201209224108127](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209224108127.png)

写入数据，被阻挡：

![image-20201209224133121](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209224133121.png)

**补充：**

- blocks.write：设置索引写的权限

- blocks.read：设置索引读的权限

- blocks.read_only：设置索引只可读的权限

  

## 四、查看索引

### 4.1、ElasticSearch Head 插件查看

![image-20201209224447689](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209224447689.png)

![image-20201209224507372](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209224507372.png)

### 4.2、请求方式查看

```
GET 索引名称/_settings

如：
GET index/_settings
```

![image-20201209224643528](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209224643528.png)

**查看多个索引信息：**

```
GET 索引名称1,索引名称2,.,.,./_settings

如：
GET index,test/_settings
```

![image-20201209224945904](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209224945904.png)

**查看所有索引信息：**

```
GET _all/_settings
```

![image-20201209225112148](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225112148.png)



## 五、删除索引

### 5.1、ElasticSearch Head 插件删除索引

![image-20201209225223306](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225223306.png)

### 5.2、请求方式删除索引

```
DELETE 索引名称

如：
DELETE test
```

![image-20201209225310231](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225310231.png)

![image-20201209225341538](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225341538.png)



## 六、索引Open/Close

### 6.1、关闭索引

```
POST 索引名称/_close

如：
POST index/_close
```

![image-20201209225758229](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225758229.png)

![image-20201209225830938](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225830938.png)

### 6.2、打开索引

```
POST 索引名称/_open

如：
POST index/_open
```

![image-20201209225852728](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225852728.png)

![image-20201209225910220](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209225910220.png)

**补充：**

- 关闭多个/打开多个索引，使用“,”隔开即可，如查看多个索引的方式
- 关闭所有索引/打开所有索引，使用  **_all**  即可

## 七、复制索引

```
POST _reindex
{
	"source": {"index","被复制索引名称"},
	"dest": {"index":"复制成的索引名称"}
}

如：
POST _reindex
{
  "source": {"index":"index"},
  "dest": {"index":"index_copy"}
}
```

![image-20201209230247687](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230247687.png)

![image-20201209230302216](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230302216.png)

从上图我们看出，复制的索引与被复制索引分片数与副本数不一样。其实复制索引就是复制索引的数据：

![image-20201209230432294](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230432294.png)

## 八、索引别名

### 8.1、ElasticSearch Head 插件修改索引别名

![image-20201209230543818](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230543818.png)

![image-20201209230557238](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230557238.png)

![image-20201209230613029](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230613029.png)

**删除索引别名：**

![image-20201209230636765](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230636765.png)



### 8.2、请求方式操作索引别名

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "索引名称",
        "alias": "索引别名"
      }
    }
  ]
}

如：
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "index_copy",
        "alias": "index_copy_aliases"
      }
    }
  ]
}
```

![image-20201209230806639](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230806639.png)

![image-20201209230909703](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209230909703.png)

**删除索引别名：**(修改 请求中的**add**改为**remove**即可)

```
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "索引名称",
        "alias": "索引别名（起好的别名）"
      }
    }
  ]
}

如：
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "index_copy",
        "alias": "index_copy_aliases"
      }
    }
  ]
}
```

![image-20201209231109420](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209231109420.png)

### 8.3、查看索引别名

```
GET /索引名称/_alias

如：
GET /index_copy_aliases/_alias
```

![image-20201209231338107](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209231338107.png)

**查看别名对应的索引：**

```
GET /索引名称/_alias

如：
GET /index_copy/_alias
```

![image-20201209231528792](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209231528792.png)

**查看集群上所有可用别名：**

![image-20201209231723629](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209231723629.png)

```
GET /_alias
```

![image-20201209231742108](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209231742108.png)