# 作用
该语句的作用是为了在查询时，避免其他用户对该表进行插入，删除，修改等操作，造成表的不一致性。**在查询时增加FOR UPDATE字段，会给数据表增加排他锁，即悲观锁**。

给你举几个例子：
- select * from t for update 会等待行锁释放之后，返回查询结果
- select * from t for update nowait 不等待行锁释放，提示锁冲突，不返回结果
- select * from t for update wait 5 等待5秒，若行锁仍未释放，则提示锁冲突，不返回结果
- select * from t for update skip locked 查询返回查询结果，但忽略有行锁的记录

