---
typora-root-url: images

---

# MongoDB数据库

![](/MongoDB.png)

## 一、MongoDB数据入门

### 概览

NoSQL数据库：基于分布式文件存储的数据库；

#### 优点：

- 高性能、高并发
- 灵活的数据模型
- 便于横向扩展
- 自动分片存储
- 支持分布式查询
- 继承内存缓存  

#### SQL与MongoDB对应术语：

|    SQL术语    |        MongoDB术语         |
| :-----------: | :------------------------: |
|   database    |          database          |
|     table     |         collection         |
|      row      |          document          |
|    column     |           field            |
|     index     |           index            |
|  table joins  |          $lookup           |
| primary joins |        primary key         |
|  aggrefation  |    aggregation pipeline    |
| transactions  | multi-document transaction |

### 历史版本特性

![历史版本](/MongoDB历史版本.png)

### 架构

![架构](/MongoDB架构.png)

#### MongoDB主从复制架构

![MongoDB主从复制架构](/MongoDB主从复制架构.png)

#### MongoDB分片集群架构

![MongoDB分片集群架构](/MongoDB分片集群架构.png)

#### 阿里云MongoDB架构

![阿里云MongoDB架构](/阿里云MongoDB架构.png)

### 下载安装

#### 官网地址

[https://www.mongodb.com/download-center/enterprise](https://www.mongodb.com/download-center/enterprise)

根据操作系统以及位数选择合适的安装包进行下载；

#### 安装

在Windows系统下安装时如果像跟其他的软件安装一样一直Next下去的话在进度条在一定时间后会停止不动到很久很久，因此需要修改一些内容：

##### 安装界面

选择“Custom”安装方式，然后Next；

![MongoDB安装](/MongoDB安装1.png)

##### 调整安装路径

点击Browser，选择安装目录，根据需要自行选择；

![MongoDB安装](/MongoDB安装2.png)

##### 注意页面

去掉左下角“Install MongoDB Compass”的勾选，然后Next；（不去掉的话会在安装过程中停顿特别久）

![MongoDB安装](/MongoDB安装3.png)

##### 配置MongoDB

在MongDB\data目录下创建存放数据库文件的文件夹db（因为启动MongoDB服务之前必须创建数据库文件的存放文件夹，否则命令不会自动创建，而且无法成功启动）

##### 启动MongoDB服务（服务器端）

在MongDB\bin目录下启动Cmd（即进入bin文件夹，然后在文件路径处输入cmd再回车），然后输入命令`mongod --dbpath dbPath`,dbPath指上面创建的db文件夹的路径；

![Cmd](/Cmd启动1.png)

![cmd](/Cmd启动2.png)

##### 访问

启动成功后在浏览器输入网址：http://localhost:27017/ 即可看到如下输出：

![输出](/访问输出.png)

### Shell连接及基本操作

#### 连接（客户端）

启动mongoDB服务后，在MongDB\data目录进入cmd，然后输入以下命令：`mongo -port 27017`连接mongo（端口号不指定默认27017），进入MongoDB Enterprise模式。

![连接](/MongoDB命令行模式.png)

#### 基本操作

在MongoDB Enterprise模式下，进行MongoDB的命令输入:

```shell
# MongoDB中数据库和表结构无需预先定义，同时可存储的字段可以不一致。
# 帮助，显示常用指令
help
# 查看存在的数据库
show dbs
# 查看数据库状态
db.stats()
# 进入数据库
use mydb
# 查看该数据库中存在的表
show collections
# 往表中插入数据
db.mydb.insert({id:12,name:'dbh'})
# 保存数据
db.mydb.save({id:523,age:32})
# 查找数据
db.mydb.find()
# 计数
db.mydb.find().count()
# 排序
db.mydb.find().sort({age:-1})  逆序输出
# 美观输出
db.mydb.find().pretty()
# 删除数据库
db.dropDatabase()
# 删除集合
db.collection.drop()
# 创建索引
db.orders.ensureIndex({"title":1})
```

## 数据查询与聚合分析

> input->查询过滤->清洗去重->聚合分组->output

### MongoDB条件查询

#### 与sql查询语句对比

![对比](/mongo语法对比.png)

#### Mongo运算符

![运算符](/Mongo运算符.png)

#### 正则用法

```shell
# 包含
db.users.find({name:/.* a .*/})
# 以a开头
db.users.find({name:/^a.*/})
# 以a结束
db.users.find({name:/.* a$/})
```

#### 嵌套文档条件查询

```shell
# 插入数据
db.users.insert{name:"zhang",buy:[{title:"iphone",price:6000}, {title="macbook",price:10000}]}
# 查询
db.orders.find({"buy.title":"iphone"})
```

### 映射Project与过滤

MongoDB数据查询与结果投影：

返回所有字段：

- db.users.find()
- db.users.find({age:{$gte:18}})

返回特定字段：

- db.users.find()
- db.users.find({age:{$gte:18}},{name:1,age:1}) :返回age>=18的所有条目的name和age列；

排除特定字段：

- db.users.find()
- db.users.find({age:{$gte:18}},{name:0,age:0})：返回age>=18的所有条目（条目中不含name和age列）；

### 去除重复Distinct

查询所有数据去重：

- db.users.distinct("name")
- db.users.distinct("item.count")

条件查询后去重：

- db.users.distinct("name",{age:18})
- db.users.distinct("item.count",{"age":18})

等价命令：

- db.runCommand({distinct:"orders",key:"name"})

### 分组统计Group

#### 聚合函数

db.collection.aggregate()+$group或者db.collection.mapReduce()

```shell
# 组合格式
  $group:
    {
      _id: <expression>,  # 由该表达式进行组合
      <field1>: { <accumulator1> : <expression1> }, 
      ...
    }
    
db.users.aggregate([
{$match:{ name:/.* a.*/}}, # 查找
{$group:{ _id:"$name",total={$sum:"$prices"}}} # 组合
])
```

#### MapReduce

```shell
# 先查询--> 再map--> 然后reduce --> 输出
db.users.mapReduce(
	function(){emit(this.name,this.age)},   # map
	function(key,values){return Array.sum(values)},   # reduce
	{
		query:{name:/.*a.*/} ,       # 查询
		out:order_totals        # 输出
	}
)
```

#### 聚合管道

## MongoDB核心知识

### 数据逻辑架构

![mongoDB数据逻辑架构](/mongoDB数据逻辑架构.png)

### 存储引擎

![存储引擎](/mongo存储引擎.png)

存储引擎负责管理数据存储，包括内存和磁盘两个区域；

MongoDB官方支持：

- MMAPv1引擎：3.2版本之前默认
- WiredTiger引擎：3.2版本之后默认
- Encrypted引擎：企业版支持，加密引擎
- In-Memory引擎：企业版支持

阿里云MongoDB版扩展支持的存储引擎：

- RocksDB引擎：Facebook开源的NoSQL存储引擎
- TerarkDB引擎：在提高压缩率的同时，大幅提高随机查询的性能

如何查看已安装的MongoDB引擎：`db.serverStatus().storageEngine`

### 集合Collection

![集合](/MongoDB集合.png)

查询Collections信息：`db.collections.stats()`

### 文档模型与BSON 

![BSON](/MongoDB文档模型.png)

MongoDB文档模型设计原则：

- Normalized范式：
- Denormalized反范式：

MongoDB文档结构：

- 引用方式：也就是RDBMS中的外键关联
- 嵌入式：也就是Ducoment文档中的内嵌节点

### 