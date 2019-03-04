## nosql not only sql

[TOC]

#### MongoDB

文档型非关系型数据库,存储数据格式采用BSON.强一致性，事物机制。

CRUD 全文检索

##### 常用命令

```
show dbs； 查看当前数据库
```

```
show tables；查看当前集合
```

```
use dbname； 切换数据库
```

```
db.friend.insertOne({name:"xx"})； 引用不存在的数据库 再插入一条数据才会创建数据库 不指定ID
db.friend.insertOne({name:"xx"，"friends":{name:"xx"}})； 嵌套插入
db.friend.insertOne({_id:21323，name:"xx"})；不指定id
db.friend.insertMany([{name:"xx"}，{name:"xx"}])；插入多条或
db.friend.insert([{name:"xx"}，{name:"xx"}])；
```

```
db.friend.drop()；删除集合
```

```
db.dropDatabase()；  删除当前数据库
```

```
db.friend.find() 查询全部数据
db.friend.find({name:"xx"})查询name=xx的数据
db.friend.find({name:"xx"}，{name:"x1"}) 查询 name=xx和x1的数据
运算符
$and 并 $or 或 $gte  大于等于
db.friend.find({$and:[{name:"xx"}，{name:"x1"}])  查询 name=xx和x1的数据
db.friend.find({name:{$in:["xx"，"x1"]}})查询 name=xx和x1的数据
```

```
sort排序 skip limit 从第几行开始到第几行的数据

db.friend.find.sort({salary:1}).skip(5).limit(5);按照salary正序
```

```
嵌套查询
db.friend.find({"friend.name":"xx"});
db.friend.find({"name":["redis"，"zookeeper"，"dubbo"]});
db.friend.find({"name":"redis"})
db.friend.find({"friend":{$in:{name:"redis"}}})

db.friend.find({"friend":{$elemMatch:{name:"redis","age":20}}})必须完整匹配
db.friend.find({"friend.name":"redis","friend.age":20})不需要完整匹配
```

```
db.friend.createIndex(name:"text") 创建文本索引 
db.friend.find({$text:{$search:"xxx"}}) 全文检索 采用倒排索引
db.friend.find({$text:{$search:"\"xxx\""}})一整段字符串查询
db.friend.find({$text:{$search:"xxx -j"}}) 包含xxx 不包含 j
```

