# 说明

第一块做架构的介绍，原因：

* 首先要对mysql有一个则很难个体的意识
* 其次我们能对mysql里面的一些概念更加理解，帮组我们做优化的时候分析问题
* 知道我们优化的是mysql内部的哪些部分

# 1 Mysql 整体架构图

![20170320171720213](E:\文档\study-note\study-note.assets/20170320171720213.png)

我们平时使用的都是上面的Client层，也就是客户端

要了解Mysql的性能优化，那么我们需要知道的是Mysql的服务端怎么运作的，以及它的存储引擎。

# 2 细化的流程交互图

# 3 Mysql中的索引

**索引是高效获取数据的数据结构，在数据存储系统中索引以文件形势存在。**

这里是一部分，后续更新之后有没有新的我不知道

* hash :  不建议使用，原因：
  * 因为hash容易产生hash冲突
  * **无法做范围查询，只能做等值查询的索引，不符合实际运用情况**, 比如：select * from table where id > 1

* FullText ：类似全文检索的索引

* R Tree:

* B Tree:

* B+Tree: mysql的InnoDB和MYSAM引擎都是走的 B+Tree 索引

  

**什么是执行计划？**
执行计划 ： 数据库根据SQL语句和相关表的统计信息作出的查询方案，该方案由查询优化器自动解析产生
比如，一条SQL语句是从10条数据中查询一条数据，正常情况 查询优化器 会选择索引查找的方
式。  如果此时你的SQL语句使用不当 那么很可能就要进行全表扫描了

**怎么看执行计划？**

在select语句之前 加上`explain`:

比如：`EXPLAIN select * from goods where price = 70`

增删改语句也可以变相的使通过`explain`来查看执行计划，因为增删改也是带有查询语句的，比如

`update goods set price = 100 where goodId=2 `

这时候我们可以通过`EXPLAIN select * from goods where goodId=2 `,来查看执行计划，看是否按照你所想要的结果高效执行

**那么 sql 查询中 为了提高查询效率 ， 需要注意的什么呢？**

通过查看执行计划总结得出：

1. 首先需要写出 统一 的 sql 语句 举例：
    * select * from tablename
    * select * FROM tablename

    虽然只是大小写不同 ，但是查询分析器 就认为这是两个不同的sql 语句，他需要去解析两次，生成两个执行计划

    所以我们 在写程序的时候 应该去保证同样的 SQL语句 在任何地方都是一致的。多一个空格都不可以

2. 尽量不要把 SQL语句 写的太长，太过冗余

     根据经验嵌套超过三层，查询优化器就很容易给出错误的执行计划。另外执行计划是可以复用的，越是简单的SQL语句复用的可能性越高。 而复杂的SQL语句只要有一个字符发生变化，就必须重新解析。然后这样的垃圾信息，还会被缓存在内存中，可想而知，数据库的性能会受到多大影响

3. 合理使用 like 模糊查询

     like '%xxx' 这样的 % 号出现在关键字前面的模糊查询 肯定会走全表扫描，因此 除非必要，否则不要在关坚持前面加%。

4. 应该尽量避免在 where 字句中 对字段进行 null 值得判断 不管是 is null / is not null

   比如 : select id from table where name is null     假设 这个表我们对 name 设置
   了索引，但是 查询分析器不会使用索引，因此查询效率低下。为了避免这样的情况发生，我们应该设置表的值为0或'’根据你字段类型来设置一个值，不管怎么也别是null.这样就可以 通过 判断 name = 0 或则是 name = ...‘’来过滤了

5. 避免在 where 字句中 使用 不等于 ！= 或则 <> 这样的操作符

     比如 : select id from table where name != 0

     使用这种查询条件操作符 不会使用索引，而 > , >= , < , <= , = , between and ,这样的操作符，数据库才会使用
     索引。那么该sql 应该写成 select id from table where name > 0 union all select id from table where name <0

6. 为什么 上述5的 优化后sql 不适用 or 呢 ， 因为 我们也需要避免在 SQL语句中 使用 or 来连接条件

7. 少用 in 或 not in  ，在子查询当中用 exists 代替 in 是一个好的选择

8. 避免在 where 字句中进行 函数 或 表达式 操作

    比如：

    * select id from table where name/2 = 50
    * select id from table where substring(name,1,8) = 'xxxx'

9. 在子查询当中 用 exists 代替 in 是一个好的选择

10. count(*) 这样不带任何条件的 count 会导致全表扫描

    
