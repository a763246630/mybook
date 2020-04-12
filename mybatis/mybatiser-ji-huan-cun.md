## Mybatis的两级缓存

[TOC]

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

#### **挽救二级缓存？**

想更高效率的使用二级缓存是解决不了了。

但是解决多表操作避免脏数据还是有法解决的。解决思路就是通过拦截器判断执行的sql涉及到那些表（可以用jsqlparser解析），然后把相关表的缓存自动清空。但是这种方式对缓存的使用效率是很低的。

设计这样一个插件是相当复杂的，既然我没想着去实现，就不废话了。



#### **核心类**

SqlSessionFactoryBuilder：

 用于构建会话工厂，基于 config.xml environment 、props 构建会话工厂,构建完成后即可丢弃。

SqlSessionFactory：

用于生成会话的工厂，作用于整个应用运行期间，一般不需要构造多个工厂对像

SqlSession：

作用于单次会话，如WEB一次请求期间，不能用作于某个对像属性，也不能在多个线程间共享，因为它是线程不安全的。



#### **接口式编程**

由于每次调用时都去找对应用 statement 以及拼装参数，使用上不是特别友好，myBatis 引入了接口的机制，将接口与mapper.xml  的namespace 名称绑定，MyBatis就可以根据ASM工具动态构建该接口的实例。

mapper 映射器接口实例

通过 session.getMapper(Class<T> type) 就可以获取mapper 实例，该实例一般作用于方法域。

#### **设置**



| 设置参数                  | 描述                                                         | 有效值                                                       | 默认值                                                       |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| cacheEnabled              | 该配置影响的所有映射器中配置的缓存的全局开关                 | true \| false                                                | true                                                         |
| lazyLoadingEnabled        | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置fetchType属性来覆盖该项的开关状态 | true \| false                                                | false                                                        |
| aggressiveLazyLoading     | 当启用时，对任意延迟属性的调用会使带有延迟加载属性的对象完整加载；反之，每种属性将会按需加载。 | true \| false                                                | true                                                         |
| multipleResultSetsEnabled | 是否允许单一语句返回多结果集（需要兼容驱动）。               | true \| false                                                | true                                                         |
| useColumnLabel            | 使用列标签代替列名。不同的驱动在这方面会有不同的表现， 具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。 | true \| false                                                | true                                                         |
| useGeneratedKeys          | 允许 JDBC 支持自动生成主键，需要驱动兼容。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作（比如 Derby）。 | true \| false                                                | False                                                        |
| autoMappingBehavior       | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                                      |
| defaultExecutorType       | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                       |
| defaultStatementTimeout   | 设置超时时间，它决定驱动等待数据库响应的秒数。               | Any positive integer                                         | Not Set (null)                                               |
| defaultFetchSize          | Sets the driver a hint as to control fetching size for return results. This parameter value can be override by a query setting. | Any positive integer                                         | Not Set (null)                                               |
| safeRowBoundsEnabled      | 允许在嵌套语句中使用分页（RowBounds）。                      | true \| false                                                | False                                                        |
| mapUnderscoreToCamelCase  | 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。 | true \| false                                                | False                                                        |
| localCacheScope           | MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。 | SESSION \| STATEMENT                                         | SESSION                                                      |
| jdbcTypeForNull           | 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType enumeration. Most common are: NULL, VARCHAR and OTHER | OTHER                                                        |
| lazyLoadTriggerMethods    | 指定哪个对象的方法触发一次延迟加载。                         | A method name list separated by commas                       | equals,clone,hashCode,toString                               |
| defaultScriptingLanguage  | 指定动态 SQL 生成的默认语言。                                | A type alias or fully qualified class name.                  | org.apache.ibatis.scripting.xmltags.XMLDynamicLanguageDriver |
| callSettersOnNulls        | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这对于有 Map.keySet() 依赖或 null 值初始化的时候是有用的。注意基本类型（int、boolean等）是不能设置成 null 的。 | true \| false                                                | false                                                        |
| logPrefix                 | 指定 MyBatis 增加到日志名称的前缀。                          | Any String                                                   | Not set                                                      |
| logImpl                   | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | Not set                                                      |
| proxyFactory              | 指定 Mybatis 创建具有延迟加载能力的对象所用到的代理工具。    | CGLIB \| JAVASSIST                                           | JAVASSIST (MyBatis 3.3 or above)                             |

设置MyBatis 全局参数，约定myBatis 的全局行为

```
<settings>

<!-- 开启二级缓存-->

  <setting name="cacheEnabled" value="true"/>

  <!-- 开启驼峰命名适配-->

  <setting name="mapUnderscoreToCamelCase" value="true"/>

//none:不自动映射*	*partial:只会自动映射，没有定义嵌套结果集的结果*	*full: 不管是否嵌套，都自动映射*

	<setting name="autoMappingBehavior" value="none"/>

<settings>
```

示例驼峰命名开启与关闭：

l  尝试开关 mapUnderscoreToCamelCase 属性 来观察Account 数据查询情况。

约定规则：

l mapper 中的 namespace必须与对应的接口名称对应。

l 通过 class 或package 中加载时 .xml 文件必须与接口在同一级目录。

#### **类型处理器**

持久层框架其中比较重要的工作就是处理数据的映射转换，把java 类型转换成jdbc 类型的参数，又需要把jdbc 类型的结果集转换成java 类型。在mybatis 中是通过 TypeHandler 接口来实现的。

```
<select id="selectById"        <!-- 语句块的唯一标识 与接口中方法名称对应 -->

  parameterType="User"   <!--参数java类型-->

  resultType="hashmap"   <!--返回结果java类型-->

  resultMap="userResultMap" <!--返回结果映射-->

  flushCache="false"      <!--true 每次调用都会刷新 一二级缓存-->

  useCache="true"         <!--true 是否保存至二级缓存当中去-->

  timeout="10"

  statementType= PREPARED">

你可以设置自定义处理器
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"  />
</typeHandlers>
```

```
可以通过以下两种方式指定处理的范围
javaType="long", jdbcType="Date"
@MappedJdbcTypes( jdbc类型) @MappedTypes       java类型
示例：
long 类型时间戳转换成 日期类型

添加算定义处理类：
@MappedJdbcTypes(JdbcType.TIMESTAMP)
@MappedTypes(Long.class)
public class LongTimeHandler extends BaseTypeHandler<Long> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Long parameter, JdbcType jdbcType) throws SQLException {
        ps.setDate(i, new Date(parameter));
    }

    @Override
    public Long getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return rs.getDate(columnName).getTime();
    }

    @Override
    public Long getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return rs.getDate(columnIndex).getTime();
    }

    @Override
    public Long getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return cs.getDate(columnIndex).getTime();
    }
}

在resultMap中指定 typeHandler：
<resultMap id="account2" type="com.tuling.mybatis.dao.Account">
      <result property="createTimestamp" column="createTimestamp"
              typeHandler="com.tuling.mybatis.dao.LongTimeHandler"/>
  </resultMap>
  <select id="selectById2" resultMap="account2">
  select a.*,a.createTime as createTimestamp from account a where id = #{id}
</select>
```

嵌套结果映射

关联 association 

示例：

```
<resultMap id="accountAndUser" type="com.tuling.mybatis.dao.Account">

   <id property="id" column="id"/>

    <association property="user" javaType="com.tuling.mybatis.dao.User">

      <id property="id" column="user_id"/>

     <result property="name" column="userName"/>

   </association>

</resultMap>

<select id="selectAccountAndUser" resultMap="accountAndUser">

   SELECT a.*, b.name userName from account a,user b where a.user_id=b.id

</select>
```

 

引入外部Select

<!--基于多次查询拼装引入 -->



```
<resultMap id="accountAndUser2" type="com.tuling.mybatis.dao.Account">

   <id property="id" column="id"/>

    <association property="user" javaType="com.tuling.mybatis.dao.User" select="selectUser"

                column="user_id">

   </association>

</resultMap>

 

<select id="selectUser" resultType="com.tuling.mybatis.dao.User">

   select * from user  where id = #{id}

</select>
```

集合collection 



#### **别名**

在myBatis 中经常会用到 java 中类型，如sql 块中中 parameterType  参数引用中 javaType 结果集映射的javaType ,都要使用java 全路径名，可以通过 

```
<typeAliases>

   <typeAlias type="com.tuling.mybatis.dao.Account" alias="account"/>

   <package name="com.tuling.mybatis.dao"  />

</typeAliases>
```

提示：建议不要设置。因为常用的类 mybatis 已经内置别名，而自定义的类设置别反而不好去找，影响阅读。



工作原理解析
mybatis应用程序通过SqlSessionFactoryBuilder从mybatis-config.xml配置文件（也可以用Java文件配置的方式，需要添加@Configuration）来构建SqlSessionFactory（SqlSessionFactory是线程安全的）；

然后，SqlSessionFactory的实例直接开启一个SqlSession，再通过SqlSession实例获得Mapper对象并运行Mapper映射的SQL语句，完成对数据库的CRUD和事务提交，之后关闭SqlSession。

说明：SqlSession是单线程对象，因为它是非线程安全的，是持久化操作的独享对象，类似jdbc中的Connection，底层就封装了jdbc连接。

详细流程如下：

1、加载mybatis全局配置文件（数据源、mapper映射文件等），解析配置文件，MyBatis基于XML配置文件生成Configuration，和一个个MappedStatement（包括了参数映射配置、动态SQL语句、结果映射配置），其对应着<select | update | delete | insert>标签项。

2、SqlSessionFactoryBuilder通过Configuration对象生成SqlSessionFactory，用来开启SqlSession。

3、SqlSession对象完成和数据库的交互：
a、用户程序调用mybatis接口层api（即Mapper接口中的方法）
b、SqlSession通过调用api的Statement ID找到对应的MappedStatement对象
c、通过Executor（负责动态SQL的生成和查询缓存的维护）将MappedStatement对象进行解析，sql参数转化、动态sql拼接，生成jdbc Statement对象
d、JDBC执行sql。

e、借助MappedStatement中的结果映射关系，将返回结果转化成HashMap、JavaBean等存储结构并返回。
 