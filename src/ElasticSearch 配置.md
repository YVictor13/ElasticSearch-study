# ElasticSearch 的配置

```
Elasticsearch 的配置同样遵循着“约定大于配置”的设计原则，用户可以选择使用群集更新设置API在正在运行的群集上更改大多数配置，也可
以选择通过配置文件对Elasticsearch 进行配置。
```

## 一、配置文件位置信息

```
在ElasticSearch中有三个配置文件，分别为（默认位置 config目录下）elasticsearch.yml、jvm.options和log4j2.properties。如下图：
```

![image-20201205151729727](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201205151729727.png)

**说明**

- elasticsearch.yml：配置ElasticSearch
- jvm.options：配置ElasticSearch依赖的JVM信息
- log4j2.properties：配置ElasticSearch日志记录中的各个属性



## 二、配置文件格式信息

```
ElasticSearch的配置文件为yaml。常见配置方式：
```

- 层级方式

  ```
  path:
  	data:/var/lib/es
  	logs:/var/log/es
  ```

- 单条方式

  ```
  	path.data:/var/lib/es
  	path.logs:/var/log/es
  ```

- 引用环境变量方式

  ```
  node.name:${HOSTNAME}
  network.host:${ES_NETWORK_HOST}
  ```

  

## 三、配置JVM选项

```java
一般情况下，Elasticsearch中很少修改JVM选项，最有可能更改配置堆的大小；
默认情况下，ElasticSearch配置JVM使用最小堆空间和最大堆空间大小均为1G

在ElasticSearch中，我们通过配置jvm.options文件中的xms(最小堆大小)和xmx(最大堆大小)两个参数来指定整个堆大小，一般情况下，两
者参数设置应为相等。
```

**jvm.options配置文件说明**

- 以 “**#**”开头的行被视为注释并忽略

- 以“**-**”开头的行被视为独立于本机**JVM**版本号的**JVM**选项

  ```
  -Xmx2g
  ```

- 以“**数字：**”开头的行被视为一个**JVM**选项，该选项仅在本机**JVM**的版本号相互匹配时适用

  ```
  8:-Xmx2g
  ```

- 以“**数字-**”开头的行被视为一个**JVM**选项，该选项仅在本机JVM的版本号大于或等于该数字版本号时才适用

  ```
  8-：-Xmx2g
  ```

- 以“**数字-数字**”开头的行被视为一个**JVM**选项，该选项仅在本机**JVM**版本号在这两个数字版本号范围内时适用

  ```
  8-9：-Xmx2g
  ```

- 空白行忽略即可，其他都被拒绝解析

  

## 四、安全配置

```
由于ElasticSearch中，部分设置信息是敏感而需要保密的，仅仅通过文件系统权限来保护这些信息是不够的，所以需要配置安全维度的信息。
- ElasticSearch中提供了一个密钥库和相应的密钥库工具来管理密钥库中的配置。
- 对密钥库所做的配置修改，皆在重启后有效
- 安全配置需要在集群中的每个节点上指定，而且所有安全配置都是特定于节点的配置，所以每个节点上必须有相同的值。
```

**安全配置常规操作**

- 创建密钥库

  创建密钥库（**elasticsearch.keystore**）命令（如下）。创建完后，可生成 **elasticsearch.yml**和**elasticsearch.keystore**两个文件

  ```
  bin/elasticsearch-keystore create
  ```

- 查看密钥库中的设置列表

  查看密钥库中的设置列表命令（如下）

  ```
  bin/elasticsearch-keystore list
  ```

- 添加字符串设置

  设置敏感的字符串命令（如下），执行完命令后，提示输入设置值。

  ```
  bin/elasticsearch-keystore add the.setting.name.to.set
  ```

  同时用户可以使用**--stdin**标志在窗口**stdin**中输出待设置的目标值。

  ```
  cat /file/containing/setting/value | bin/elasticsearch-keystore add --stdin the.setting.name.to.set
  ```

- 添加文件设置

  用户可以使用添加文件命令添加敏感信息文件。配置时需确保将文件路径作为参数包含在设置名称之后。

  ```
  bin/elasticsearch-keystore add-file the.setting.name.to.set /path/example-file.json
  ```

- 删除密钥设置

  从密钥库中删除设置命令（如下）

  ```
  bin/elasticsearch-keystore remove the.setting.name.to.remove
  ```

- 可重新加载的安全配置

  由于配置**elasticsearch.keystore**文件，和配置**elasticsearch.yml**文件一样，对密钥库内容的更改不会自动应用于正在运行的**elasticsearch**节点上，所以需要重新启动节点才能重新读取设置；

  对于一些安全设置，我们可以标记位可重新加载，这样就可以在正在运行的节点上重新读取和应用；

  所有安全设置的值（无论是否可重新加载），在所有集群节点上必须相同更改所需的安全设置，才可使用命令(如下)进行重新加载、读取和应用。

  ```
  POST _nodes/reload_secure_settings
  ```

  该API接口将解密并重新读取每个集群节点上的整个密钥库，但只限于可重载的安全配置，其他设置更改需要在重启之后生效。

  **注**：

  ```
  当更改多个可重新加载的安全设置时，用户需要在每个集群节点上都修改所有配置，然后再调用重新加载安全配置API，不是在每次修改后就重
  新加载。
  ```

  

## 五、日志记录配置

```
在ElasticSearch中，使用log4j2来记录日志。所以我们可以通过log4j2.properties文件配置log4j2；
```

**ElasticSearch中相关的三个公开属性信息：**

- $sys:es.logs.base_path：日志文件目录地址
- $sys:es.logs.cluster_name：集群名称（默认配置中，用作日志文件名的前缀）
- $sys:es.logs.node_name：节点名称（如果显式地设置了节点名称）



**log4j2.properties文件的配置信息：**

```java
# 配置RollingFile的appender属性
appender.rolling.type = RollingFile 
appender.rolling.name = rolling 
# 日志信息将输出到 ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.json中
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}_server.json
# 使用json文件格式输出
appender.rolling.layout.type = ESJsonLayout
# type_name是填充ESJsonLayout的类型字段的标志，该字段可以让我们在解析不同类型的日志时更加简单
appender.rolling.layout.type_name = server
# 日志将会滚动输出到${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-
%i.json.gz中，日志文件会被压缩，且呈i递增状态
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-
%d{yyyy-MM-dd}-%i.json.gz  
appender.rolling.policies.type = Policies
# 使用基于时间戳的新增日志滚动策略    
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy
# 按天滚动新增日志    
appender.rolling.policies.time.interval = 1
# 在日期时间上对齐标准，而不是按每24小时来新增一次滚动日志文件    
appender.rolling.policies.time.modulate = true
# 按日志文件大小的策略来滚动新增日志文件    
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy
# 每生成128MB的日志文件，就滚动新增一次日志    
appender.rolling.policies.size.size = 128MB 
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.fileIndex = nomax
# 每次新增滚动日志时执行删除日志文件动作   
appender.rolling.strategy.action.type = Delete
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path}
# 仅当文件匹配时才删除日志文件
appender.rolling.strategy.action.condition.type = IfFileName
# 该配置仅用于删除日志文件    
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-*
# 只有当日志目录下积累了较多日志时才删除    
appender.rolling.strategy.action.condition.nested_condition.type = IfAccumulatedFileSize
# 压缩日志的条件是日志文件大小达到2G    
appender.rolling.strategy.action.condition.nested_condition.exceeds = 2GB
```



**配置日志记录级别方式：**

- 通过命令行配置

  适用于：当在单个节点临时调用一个问题（如在后启动时或在开发过程中）时

  命令如下：

  ```
  -e<name of logging hierarchy>=<level> 
  如 -e logger.org.elasticsearch.transport=trace
  ```

- 通过elasticsearch.yml文件配置

  适用：当临时调用一个问题，但没有通过命令行启动ElasticSearch；或者希望在更持久的基础上调整日志级别时

  配置属性如下：

  ```
  <name of logging hierarchy>:<level>
  ```

- 通过集群配置

  适用：当需要动态调整活动运行的集群上的日志级别时

  方法如下：

  ```
  PUT /_cluster/settings
  {
      "transient":{
  		"<name of logging hierarchy>":"<level>"
      }
  }
  
  如：
  PUT /_cluster/settings
  {
      "transient":{
  		"logger.org.elasticsearch.transport":"trace"
      }
  }
  ```

- 通过log4j2.properties配置

  适用：当需要对日志程序进行细粒度的控制时（如将日志程序发送到另一个文件，或者以不同的方式管理日志程序）

  配置属性如下：

  ```
  logger.<unique_identifier>.name = <name of logging hierarchy>
  logger.<unique_identifier>.level = <level>
  
  如：
  logger.<unique_identifier>.name = logger.org.elasticsearch.transport
  logger.<unique_identifier>.level = trace
  ```



## 六、JSON日志格式

```java
# 使用json文件格式输出
appender.rolling.layout.type = ESJsonLayout
# type_name是填充ESJsonLayout的类型字段的标志，该字段可以让我们在解析不同类型的日志时更加简单
appender.rolling.layout.type_name = server

日志默认为JSON格式打印，可通过如上两个属性进行配置，如需使用自定义布局，则配置appender.rolling.layout.type即可。
例如：
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}][%node_name]%marker %.-10000m%n
```



 [上一篇：Elasticsearch 安装指南](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/ElasticSearch%20%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97.md)

[下一篇：Elasticsearch 核心概念](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/Elasticsearch%20%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.md)