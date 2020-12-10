# ElasticSearch 安装指南
所需网站：
- [Es 中文官网](https://www.elastic.co/cn/)
- [ElasticSearch 官网](https://www.elastic.co/cn/elasticsearch/)
- [ElasticSearch 操作系统支持矩阵](https://www.elastic.co/cn/support/matrix)

## 1.单节点安装
安装步骤：
- 进入ElasticSearch 官网，根据操作系统点击下载ElasticSearch（如果操作系统非主流，请查阅ElasticSearch的操作系统支持矩阵）
  ![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122163452288.png)

- 下载成功后，将文件解压（如下）
  ![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122164109826.png)

- 在bin目录下，进入cmd，直接执行./elasticsearch 启动即可。（出现如下图所示  started ，即为启动成功）

  ![image-20201205145719274](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201205145719274.png)

- 查看节点信息。单节点默认监听9200端口，所以直接在浏览器中输入'''localhost:9200'''即可查看节点信息
  开发者可以自定义设置集群名字（默认elasticsearch）和节点名字（与个人电脑有关）
  ![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122170854109.png)

- 自定义设置节点和集群名字。进入config文件夹，打开 elasticsearch.yml文件（config/elasticsearch.yml），在文本最后加上如下配置。
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122171541997.png)
配置内容如下：
```yml
# 集群名字
culster.name: javaboy-es
# 节点名字
node.name: master
```
保存文件，重启elasticsearch（按照启动方式）,成功后，刷新浏览器（localhost:9200），即可看到集群和节点名字已改变。如下图
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122172310665.png)


## 补充：**ElasticSearch文件目录的含义**

| 目录    | 含义           |
| ------- | -------------- |
| bin     | 可执行文件目录 |
| config  | 配置文件目录   |
| jdk     | JAVA工具包     |
| lib     | 第三方依赖库   |
| logs    | 输出日志目录   |
| modules | 依赖模块目录   |
| plugins | 插件目录       |
| data    | 数据存储目录   |


## 2.HEAD插件安装
Elasticsearch-head 插件，可以通过可视化的方式查看集群信息。
### 1.浏览器插件安装
安装步骤：
- Chrom直接在Web Store搜索Elasticsearch Head，点击安装即可。
[Elasticsearch Head 下载路径](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm)
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/2020112217344170.png)
- 点击安装好的插件图标，即可看见集群和节点信息
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122174010414.png)
### 2.下载Elasticsearch-head插件安装
安装步骤：
- 进入Github项目 [Elasticsearch-head 插件Github地址](https://github.com/mobz/elasticsearch-head)
- 按照如下方式安装即可。（与单节点安装相似）
```xml
1、git clone git://github.com/mobz/elasticsearch-head.git
2、cd elasticsearch-head
3、npm install
4、npm run start
5、open http://localhost:9100/
```
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122174557929.png)
- 打开（http://localhost:9100/） 可以看到此时查看不了集群数据。（原因在于这里通过跨域的方式请求集群数据的，默认情况下，集群不支持跨域，所以这里就看不到集群数据。）
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122195642360.png)
修改 es 的 config/elasticsearch.yml 配置文件，添加如下内容，便能支持跨域，重启后，HEAD就能看到集群数据了
```xml

# 打开集群跨域通信
http.cors.enabled: true
http.cors.allow-origin: "*"

```
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122200343188.png)

## 3.分布式安装
安装步骤：
- 根据需求设置主机和从机的数量，假设1:2.
- 设置端口： master:9200,slave01:9201和slave02:9202
- 配置master，修改master的config/elasticsearch.yml配置文件，在文本最后加入如下内容，并重启master：
```yml
# 配置master
node.master: true
network.host: 127.0.0.1
```
- 配置slave01和slave02。解压两份elasticsearch.zip，分别命名为slave01和slave02
- 分别修改salve01/config/elasticsearch.yml 和 slave02/config/elasticsearch.yml，在其文末加入

```yml
// 注意：必须保持从机的集群名称和master的集群名称一致
// slave 01 
cluster.name: javaboy-es
node.name: slave01
network.host: 127.0.0.1
http.port: 9201
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]

// slave 02
cluster.name: javaboy-es
node.name: slave02
network.host: 127.0.0.1
http.port: 9202
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]

```

- 分别启动slave01和slave02
- 启动elasticsearch-head 插件，查看集群信息
![在这里插入图片描述](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/20201122210307892.png)


## 4.Kibana安装
**Kibana** 是一个 **Elastic** 公司推出的一个针对 **Elasticsearch** 的分析以及数据可视化平台，可以搜索、查看存放在 **Elasticsearch** 中的数据。
安装步骤如下：

- 下载 Kibana： [Kibana 网站](https://www.elastic.co/cn/downloads/kibana)

- 解压文件

- 配置 es 的地址信息（如果 **Elasticsearch** 是默认设置，则可以不用配置，其配置文件在 **kibana** 的 **config/kibana.yml**）

- 在**kibana** 的**bin**目录下 执行命令 ```./kibana``` 文件启动

- 浏览器 ```localhost:5601```

  ![image-20201209175834964](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209175834964.png)

- **Kibana** 安装好之后，首次打开时，可以选择初始化 **Elasticsearch** 提供的测试数据，也可以不使用。
![image-20201209175856911](https://raw.githubusercontent.com/YVictor13/ElasticSearch-study/master/image/image-20201209175856911.png)

[下一篇：ElasticSearch 的配置](https://github.com/YVictor13/ElasticSearch-study/blob/master/src/ElasticSearch%20%E9%85%8D%E7%BD%AE.md)