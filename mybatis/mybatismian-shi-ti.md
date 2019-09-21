[TOC]



#### 1.mybatis和hibernate的相同与不同点 ORM(Object Relational Mapping）

Hibernate ：Hibernate 是当前最流行的ORM框架，对数据库结构提供了较为完整的封装。

Mybatis：Mybatis同样也是非常流行的ORM框架，主要着力点在于POJO 与SQL之间的映射关系。

相同点

都是Java技术体系的ORM框架，都是对JDBC进行了封装，实现Java对象和数据库记录的映射转换。

不同点 

Mybatis不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句。mybatis可以通过XML或注解方式灵活配置要运行的sql语句，并将java对象和sql语句映射生成最终执行的sql，最后将sql执行的结果再映射生成java对象。

Mybatis易上手，直接编写原生态sql，可严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运营类软件等，因为这类软件需求变化频繁，一但需求变化要求成果输出迅速。但是灵活的前提是mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定义多套sql映射文件，工作量大，二级缓存功能不完善。

Hibernate对象/关系映射能力强，可移植性高（开发采用hql编写与数据库无关），支持大部分主流数据库（Mysql，Oracle，SqlServer，DB2等等），功能强大支持二级缓存和懒加载等，熟练使用无需自己编写sql，可提高开发效率。但是Hibernate比复杂上手难，sql优化起来难。

2.Mybatis和Hibernate的优缺点

Mybatis





