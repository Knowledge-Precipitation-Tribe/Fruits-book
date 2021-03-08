1.  Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化。
2.  如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。
3.  为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内。
4.  尽量避免在压力较大的主库上增加从库
5.  Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。
6.  为了Master的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关系为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现Slave对Master的替换，也即，如果Master挂了，可以立马启用Slave1做Master，其他不变。