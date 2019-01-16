show dbs ; 查看当前数据库

show tables；查看当前集合

use dbname； 切换数据库

db.friend.insertOne({name:"xx"}); 引用不存在的数据库 再插入一条数据才会创建数据库

db.friend.drop();删除集合

db.dropDatabase();  删除当前数据库