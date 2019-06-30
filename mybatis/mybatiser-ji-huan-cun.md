Mybatis的两级缓存

一级缓存 Session(会话)级 缓存

使用条件 

同一会话session 中

同一个方法名(mapper key)

相同的namespace和statement

失效条件

执行 session.clearCache()

两次查询之间执行 update add delete