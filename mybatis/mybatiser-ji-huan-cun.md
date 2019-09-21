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

**Cache使用时的注意事项**

1. 只能在【只有单表操作】的表上使用缓存
  不只是要保证这个表在整个系统中只有单表操作，而且和该表有关的全部操作必须全部在一个namespace下。

2. 在可以保证查询远远大于insert,update,delete操作的情况下使用缓存
  这一点不需要多说，所有人都应该清楚。记住，这一点需要保证在1的前提下才可以！

### 二级缓存 SqlSessionFactory 跨会话全局缓存

二级缓存的作用域是mapper的同一个namespace。不同的sqlSession两次执行相同的namespace下的sql语句，且向sql中传递的参数也相同，即最终执行相同的sql语句，则第一次执行完毕会将数据库中查询的数据写到缓存，第二次查询会从缓存中获取数据，不再去底层数据库查询，从而提高效率。
在MyBatis配置文件（mybatis-config.xml）中开启二级缓存(详细过程自己百度搜索开启)
//value属性默认为false
在**Mapper.xml中开启当前mapper的namespace下的二级缓存

 **避免使用二级缓存**

可能会有很多人不理解这里，二级缓存带来的好处远远比不上他所隐藏的危害。

缓存是以namespace为单位的，不同namespace下的操作互不影响。

insert,update,delete操作会清空所在namespace下的全部缓存。

通常使用MyBatis Generator生成的代码中，都是各个表独立的，每个表都有自己的namespace。
 针对一个表的某些操作不在他独立的namespace下进行。

例如在UserMapper.xml中有大多数针对user表的操作。但是在一个XXXMapper.xml中，还有针对user单表的操作。

这会导致user在两个命名空间下的数据不一致。如果在UserMapper.xml中做了刷新缓存的操作，在XXXMapper.xml中缓存仍然有效，如果有针对user的单表查询，使用缓存的结果可能会不正确。

更危险的情况是在XXXMapper.xml做了insert,update,delete操作时，会导致UserMapper.xml中的各种操作充满未知和风险。

有关这样单表的操作可能不常见。但是你也许想到了一种常见的情况。

**多表操作一定不能使用缓存**

为什么不能？

首先不管多表操作写到那个namespace下，都会存在某个表不在这个namespace下的情况。

例如两个表：role和user_role，如果我想查询出某个用户的全部角色role，就一定会涉及到多表的操作。

```
<select id="selectUserRoles" resultType="UserRoleVO">
	select * from user_role a,role b where a.roleid = b.roleid and a.userid = #{userid}
</select>
```

像上面这个查询，你会写到那个xml中呢？？

不管是写到RoleMapper.xml还是UserRoleMapper.xml，或者是一个独立的XxxMapper.xml中。如果使用了二级缓存，都会导致上面这个查询结果可能不正确。

如果你正好修改了这个用户的角色，上面这个查询使用缓存的时候结果就是错的。

这点应该很容易理解。

在我看来，就以MyBatis目前的缓存方式来看是无解的。多表操作根本不能缓存。

如果你让他们都使用同一个namespace（通过<cache-ref>）来避免脏数据，那就失去了缓存的意义。

看到这里，实际上就是说，二级缓存不能用。整篇文章介绍这么多也没什么用了。

**挽救二级缓存？**
想更高效率的使用二级缓存是解决不了了。

但是解决多表操作避免脏数据还是有法解决的。解决思路就是通过拦截器判断执行的sql涉及到那些表（可以用jsqlparser解析），然后把相关表的缓存自动清空。但是这种方式对缓存的使用效率是很低的。

设计这样一个插件是相当复杂的，既然我没想着去实现，就不废话了。
 