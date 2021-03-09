在MySQL中添加行锁可以防止不同事务版本的数据修改提交时造成数据冲突的情况。但如何避免别的事务插入数据就成了问题。

在RR级别中，事务A在update后会加锁，事务B无法插入新数据，这样事务A在update前后读的数据保持一致，避免了幻读。这个锁，就是Gap锁。

MySQL是这样实现的，在class_teacher这张表中，teacher_id是个索引，那么它就会维护一套B+树的数据关系，为了简化，我们用链表结构来表达（实际上是个树形结构，但原理相同）

![](https://tva1.sinaimg.cn/large/008eGmZEly1godinct21nj30xu0fcwfe.jpg)

如图所示，InnoDB使用的是聚集索引，teacher_id身为二级索引，就要维护一个索引字段和主键id的树状结构（这里用链表形式表现），并保持顺序排列。

Innodb将这段数据分成几个个区间

- (negative infinity, 5]
- (5,30],
- (30,positive infinity)；

update class_teacher set class_name=‘初三四班’ where teacher_id=30;不仅用行锁，锁住了相应的数据行；同时也在两边的区间，（5,30]和（30，positive infinity），都加入了gap锁。这样事务B就无法在这个两个区间insert进新数据。

行锁防止别的事务修改或删除，GAP锁防止别的事务新增，行锁和GAP锁结合形成的的Next-Key锁共同解决了RR级别在写数据时的幻读问题。