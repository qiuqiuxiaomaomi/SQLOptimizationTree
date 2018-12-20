# SQLOptimizationTree
SQL优化技术

![](https://i.imgur.com/tPBlMI3.png)

<pre>
1) 最左前缀法则
2）在索引列上做类型转换，函数变换等操作
3）存储引擎不能使用范围条件右边的列作为索引
5）!= , > , <不能使用索引
6）B-tree索引 is null不会走,is not null会走,位图索引 is null,is not null 都会走
7）like 通配符开头不能使用索引
8）尽量使用覆盖索引（只查询索引的列（索引列和查询列一致）），减少select *
9）没有使用查询条件，或者查询条件上没有建立索引
10）查询的数量是索引的大部分，30%以上
11）对小表查询
12）其他存储引擎认为使用索引反而查询性能更差时
13）in, not in ,not exist
15）在JOIN操作中（需要从多个数据表提取数据时），MYSQL只有在主键和外键的数据类型相同时才能使用索引，否则即使建立了索引也不会使用
16）在ORDER BY操作中，MYSQL只有在排序条件不是一个查询条件表达式的情况下才使用索引。尽管如此，在涉及多个数据表的查询里，即使有索引可用，那些索
引在加快ORDER BY操作方面也没什么作用
17）如果某个数据列里包含着许多重复的值，就算为它建立了索引也不会有很好的效果。比如说，如果某个数据列里包含了净是些诸如“0/1”或“Y/N”等值，就没有
必要为它创建一个索引
18）如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引
19）如果mysql估计使用全表扫描要比使用索引快,则不使用索引
20）尽量避免在where子句中使用or来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，可以
    使用 union来查询
21)如果在 where 子句中使用参数，也会导致全表扫描。因为 SQL 只有在运行时才会解析局部变
   量，但优化程序不能将访问计划的选择到运行时；它必须在编译时进行选择。然而，如果在编译时
   简历访问计划，变量的值还是未知的，因而无法作为索引选择的输入项
22)应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描
23)很多时候用 exists 代替 in 是一个好的选择
25)并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重
   复时，SQL查询可能不会去利用索引，如一表中有字段 sex，male、female几乎各一半，那么即
   使在sex上建了索引也对查询效率起不了作用。
26)尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性
   能，并会增加存储开销。这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而
   对于数字型而言只需要比较一次就够了
27)尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以
   节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些
28)避免频繁创建和删除临时表，以减少系统表资源的消耗
29)尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写
30)尽量避免大事务操作，提高系统并发能力
</pre>

为排序使用索引

![](https://i.imgur.com/msPgvCY.png)

https://blog.csdn.net/tuesdayma/article/details/81783199

<pre>
一条sql执行时间过长，如何优化，从哪些方面

1）查看sql是否涉及多表的联表查询或者子查询，如果有，看是否能进行业务拆分，相关字段冗余或者
   合并成零时表。
2）涉及联表的查询，是否能进行分表查询，单表查询之后的结果进行字段整合。
3）如果以上两种情况都不能操作，非要联表查询，那么考虑对相应的查询条件做索引，加快查询速度。
5）针对数量大的表进行历史表分离
6）数据库主从分离，读写分离，降低读写针对同一表时的压力
7）explain分析SQL语句，查看执行计划，分析索引是否用得上，分析扫描行数。
8）查看Mysql执行日志，看看是否有其他方面的问题。
</pre>

<pre>
Mysql深度分页

      一般刚开始学SQL的时候，会这样写 
         SELECT * FROM table ORDER BY id LIMIT 1000, 10; 
      但在数据达到百万级的时候，这样写会慢死 
         SELECT * FROM table ORDER BY id LIMIT 1000000, 10; 
      也许耗费几十秒
 
      网上很多优化的方法是这样的 
         SELECT * FROM table WHERE id >= (SELECT id FROM table LIMIT 1000000, 1) LIMIT 10; 
      是的，速度提升到0.x秒了，看样子还行了
      可是，还不是完美的！
 
      以下这句才是完美的！ 
          SELECT * FROM table WHERE id BETWEEN 1000000 AND 1000010; 
      比上面那句，还要再快5至10倍


      从中我们也能总结出两件事情：
         1）limit语句的查询时间与起始记录的位置成正比
         2）mysql的limit语句是很方便，但是对记录很多的表并不适合直接使用。
   
      2.对limit分页问题的性能优化方法
        利用表的覆盖索引来加速分页查询
        我们都知道，利用了索引查询的语句中如果只包含了那个索引列（覆盖索引），那么这种情况会查询很快。
        因为利用索引查找有优化算法，且数据就在查询索引上面，不用再去找相关的数据地址了，这样节省了很多时间。另外Mysql中
        也有相关的索引缓存，在并发高的时候利用缓存就效果更好了。


      另外，如果需要查询 id 不是连续的一段，最佳的方法就是先找出 id ，然后用 in 查询 
           SELECT * FROM table WHERE id IN(10000, 100000, 1000000...); 
      再分享一点
      查询字段一较长字符串的时候，表设计时要为该字段多加一个字段,如，存储网址的字段
          查询的时候，不要直接查询字符串，效率低下，应该查诡该字串的crc32或md5。


      在我们的例子中，我们知道id字段是主键，自然就包含了默认的主键索引。现在让我们看看利用覆盖索引的查询效果如何：
      这次我们之间查询最后一页的数据（利用覆盖索引，只包含id列），如下：
         select id from order limit 800000, 20 0.2秒
         相对于查询了所有列的37.44秒，提升了大概100多倍的速度

      那么如果我们也要查询所有列，有两种方法，一种是id>=的形式，另一种就是利用join，看下实际情况：
         SELECT * FROM order WHERE ID > =(select id from order limit 800000, 1) limit 20
      查询时间为0.2秒，简直是一个质的飞跃啊，哈哈

      另一种写法
         SELECT * FROM order a JOIN (select id from order limit 800000, 20) b ON a.ID = b.id
      查询时间也很短
</pre>

<pre>
查询锁表信息
      当前运行的所有事务
      select * from information_schema.innodb_trx
      当前出现的锁
      select * from information_schema.innodb_locks
      锁等待的对应关系
      select * from information_schema.innodb_lock_waits  
</pre>

<pre>
Mysql一次插入几万条数据处理方式

      1）Insert批量插入，调整max_allowed_packet
      2) 开启事务，增大innodb_log_buffer_size，增加单事务提交日志量。
      3）主键顺序插入，效率更高
      5）对要插入的数据进行分组批量插入

      INSERT INTO table (column1, column2, ..., column_n) VALUES 
      (value11, value12, ..., value1n), 
      (value21, value22, ... value2n), ..., (value_n1, value_n2, ... value_nn)
 
 
      常用的插入语句如：
      INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
             VALUES ('0', 'userid_0', 'content_0', 0);
      INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
             VALUES ('1', 'userid_1', 'content_1', 1);
      修改成：
 

      INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
      VALUES ('0', 'userid_0', 'content_0', 0), ('1', 'userid_1', 'content_1', 1);
 
      修改后的插入操作能够提高程序的插入效率。这里第二种SQL执行效率高的主要原因是合并后日志量（MySQL的binlog和innodb的事务让
      日志）减少了，降低日志刷盘的数据量和频率，从而提高效率。通过合并SQL语句，同时也能减少SQL语句解析的次数，减少网络传
      输的IO。

      
      数据有序插入。
          数据有序的插入是指插入记录在主键上是有序排列，例如datetime是记录的主键：
          INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
                 VALUES ('1', 'userid_1', 'content_1', 1);
          INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
                 VALUES ('0', 'userid_0', 'content_0', 0);
          INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
                 VALUES ('2', 'userid_2', 'content_2',2);
          修改成：
 
          INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
                 VALUES ('0', 'userid_0', 'content_0', 0);
          INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
                 VALUES ('1', 'userid_1', 'content_1', 1);
          INSERT INTO `insert_table` (`datetime`, `uid`, `content`, `type`) 
                 VALUES ('2', 'userid_2', 'content_2',2);
          由于数据库插入时，需要维护索引数据，无序的记录会增大维护索引的成本。我们可以参照InnoDB使用的B+tree索引，如果每次插
          入记录都在索引的最后面，索引的定位效率很高，并且对索引调整较小；如果插入的记录在索引中间，需要B+tree进行分裂合并等
          处理，会消耗比较多计算资源，并且插入记录的索引定位效率会下降，数据量较大时会有频繁的磁盘操作。

         
      从测试结果来看，该优化方法的性能有所提高，但是提高并不是很明显。
 
      SQL语句是有长度限制，在进行数据合并在同一SQL中务必不能超过SQL长度限制，通过max_allowed_packet配置可以修改，默认是1M，
      测试时修改为8M。
 
      事务需要控制大小，事务太大可能会影响执行的效率。MySQL有innodb_log_buffer_size配置项，超过这个值会把innodb的数据刷到
      磁盘中，这时，效率会有所下降。所以比较好的做法是，在数据达到这个这个值前进行事务提交。
</pre>

<pre>
Mysql配置优化提高SQL查询写入性能

      提高数据库插入性能的中心思想：
              1）尽量将数据一次性写入到Data File
              2）减少数据库的checkpoint 操作

      innodb_buffer_pool_size 
           如果用Innodb，那么这是一个重要变量。相对于MyISAM来说，Innodb对于
        buffer size更敏感。MySIAM可能对于大数据量使用默认的key_buffer_size也还好，
        但Innodb在大数据量时用默认值就感觉在爬了。 Innodb的缓冲池会缓存数据和索引，所
        以不需要给系统的缓存留空间，如果只用Innodb，可以把这个值设为内存的70%-80%。
        和 key_buffer相同，如果数据量比较小也不怎么增加，那么不要把这个值设太高也可以提
        高内存的使用率。

      innodb_log_file_size 
           此配置项作用设定innodb 数据库引擎UNDO日志的大小；从而减少数据库checkpoint
        操作。

           对于写很多尤其是大数据量时非常重要。要注意，大的文件提供更高的性能，但数据库恢
        复时会用更多的时间。我一般用64M-512M，具体取决于服务器的空间。

      innodb_log_buffer_size 
           此配置项作用设定innodb 数据库引擎写日志缓存区；将此缓存段增大可以减少数据库写
        数据文件次数

           默认值对于多数中等写操作和事务短的运用都是可以的。如 果经常做更新或者使用了
        很多blob数据，应该增大这个值。但太大了也是浪费内存，因为1秒钟总会 flush（这个词
        的中文怎么说呢？）一次，所以不需要设到超过1秒的需求。8M-16M一般应该够了。小的运用
        可以设更小一点。

      innodb_flush_log_at_trx_commit 
           0: Write the log buffer to the log file and flush the log file 
              every second, but do nothing at transaction commit. 
           1：the log buffer is written out to the log file at each 
              transaction commit and the flush to disk operation is performed 
              on the log file 
           2：the log buffer is written out to the file at each commit, but 
              the flush to disk operation is not performed on it 

           抱怨Innodb比MyISAM慢 100倍？那么你大概是忘了调整这个值。默认值1的意思是每一
        次事务提交或事务外的指令都需要把日志写入（flush）硬盘，这是很费时的。特别是使用
        电 池供电缓存（Battery backed up cache）时。设成2对于很多运用，特别是从MyISAM
        表转过来的是可以的，它的意思是不写入硬盘而是写入系统缓存。日志仍然会每秒flush到
        硬 盘，所以你一般不会丢失超过1-2秒的更新。设成0会更快一点，但安全方面比较差，即
        使MySQL挂了也可能会丢失事务的数据。而值2只会在整个操作系统 挂了时才可能丢数据。

      innodb_autoextend_increment 配置由于默认8M 调整到 128M
           此配置项作用主要是当tablespace 空间已经满了后，需要MySQL系统需要自动扩展多
        少空间，每次tablespace 扩展都会让各个SQL 处于等待状态。增加自动扩展Size可以减
        少tablespace自动扩展次数。

      提高数据库插入性能中心思想： 
          1、尽量使数据库一次性写入Data File 
          2、减少数据库的checkpoint 操作 
          3、程序上尽量缓冲数据，进行批量式插入与提交 
          4、减少系统的IO冲突

      提高数据库读取速度
</pre>

<pre>
Mysql表查询优化经验
</pre>

<pre>
Using index 
      查询的列被索引覆盖，并且where筛选条件是索引的是前导列

      Using where Using index
        1:查询的列被索引覆盖，并且where筛选条件是索引列之一但是不是索引的不是前导列，
          Extra中为Using where; Using index，意味着无法直接通过索引查找来查询到符
          合条件的数据
        2:查询的列被索引覆盖，并且where筛选条件是索引列前导列的一个范围，同样意味着无法
          直接通过索引查找查询到符合条件的数据
     NULL（既没有Using index，也没有Using where Using index，也没有using where）
        1，查询的列未被索引覆盖，并且where筛选条件是索引的前导列，
　　     意味着用到了索引，但是部分字段未被索引覆盖，必须通过“回表”来实现，不是纯粹地用
         到了索引，也不是完全没用到索引，Extra中为NULL(没有信息)
     Using where
        1，查询的列未被索引覆盖，where筛选条件非索引的前导列，Extra中为Using where
        2，查询的列未被索引覆盖，where筛选条件非索引列，Extra中为Using where

        using where 意味着通过索引或者表扫描的方式进程where条件的过滤，
　　    反过来说，也就是没有可用的索引查找，当然这里也要考虑索引扫描+回表与表扫描的代价。
　　    这里的type都是all，说明MySQL认为全表扫描是一种比较低的代价。
     Using index condition
        1，-- 查询的列不全在索引中，where条件中是一个前导列的范围
        2，查询列不完全被索引覆盖，查询条件完全可以使用到索引（进行索引查找）
</pre>

![](https://i.imgur.com/UCgL2Vh.png)

<pre>
Explain详细解析

      table：表名
      type：这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、
            eq_reg、ref、range、index和ALL
      possible_keys：显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关
            的域从WHERE语句中选择一个合适的语句
      key： 实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足
            的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使
            用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引
      key_len：使用的索引的长度。在不损失精确性的情况下，长度越短越好
      ref：显示索引的哪一列被使用了，如果可能的话，是一个常数
      rows：MYSQL认为必须检查的用来返回请求数据的行数

      extra列返回的描述的意义
           Distinct:一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

           Not exists: MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不
                       再搜索了

           Range checked for each Record（index map:#）:没有找到理想的索引，因此对
               于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中
               返回行。这是使用索引的最慢的连接之一

           Using filesort: 看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来
               发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行
               的行指针来排序全部行

           Using index: 列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返
                        回的，这发生在对表的全部的请求列都是同一个索引的部分的时候

           Using temporary 看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时
                 表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

           Where used 使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果
                不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有
                问题不同连接类型的解释（按照效率高低的顺序排序）

           system 表只有一行：system表。这是const连接类型的特殊情况

           const:表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因
              为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待

           eq_ref:在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取
                  一个记录，它在查询使用了索引为主键或惟一键的全部时使用

           ref:这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分
             （比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从
              表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好

           range:这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况

           index: 这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为
                 索引一般小于表数据）

           ALL:这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免
</pre>

<pre>
子查询 联合join查询效率
</pre>