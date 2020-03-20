# MyBatis相关

## 用过的MyBatis标签有哪些？
`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，
加上动态 sql 的 9 个标签，`trim|where|set|foreach|if|choose|when|otherwise|bind`等，其中为 sql 片段标签，通过`<include>`标签引入 sql 片段，<selectKey>为不支持自增的主键生成策略标签。

## #{} 和 ${} 的区别
+ `${}`是 Properties 文件中的变量占位符。它可以用于标签属性值和 sql 内部，属于静态文本替换，
比如${driver}会被静态替换为com.mysql.jdbc.Driver。
+ `#{}`是 sql 的参数占位符，Mybatis 会将SQL中的#{}替换为?号，
在 sql 执行前会使用`PreparedStatement`的参数设置方法，按序给 sql 的?号占位符设置参数值，比如`ps.setInt(0, parameterValue)`，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。

## MyBatis 延迟加载

## MyBatis一级缓存二级缓存
+ 一级缓存 是 SqlSession 级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。
+ 二级缓存 是 mapper 级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

## MyBatis中一个XML映射文件，都会写一个Dao/Mapper接口与之对应，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？
DAO/Mapper接口，对应映射文件中的`namespace`的值。接口方法名为映射文件中`MappedStatement`的id值。接口方法内的参数，就是传递给SQL的参数。<br>
Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个`MappedStatement`。<br>
DAO接口的工作原理是JDK动态代理，MyBatis运行时会使用JDK动态代理为DAO接口生成代理`proxy`对象，代理对象`proxy`会拦截接口方法，转而执行`MappedStatement`所代表的 sql，然后将 sql 执行结果返回。

