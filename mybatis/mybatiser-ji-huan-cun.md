## Mybatis的两级缓存

### 一级缓存 Session(会话)级 缓存

#### 使用条件 

同一会话session 中

同一个方法名(mapper key)

相同的namespace和statement

#### 失效条件

执行 session.clearCache()

两次查询之间执行 update add delete

#### 实现原理

都是实现了Cache接口,

DefaultSqlSession

### 二级缓存 SqlSessionFactory 跨会话全局缓存

mybatis和hibernate的相同与不同点 ORM(Object Relational Mapping）

Hibernate ：Hibernate 是当前最流行的ORM框架，对数据库结构提供了较为完整的封装。

Mybatis：Mybatis同样也是非常流行的ORM框架，主要着力点在于POJO 与SQL之间的映射关系。

相同点

1.都是orm框架