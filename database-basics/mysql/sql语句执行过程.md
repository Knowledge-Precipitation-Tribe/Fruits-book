对于下面这样一个查询语句来说他的执行顺序是什么？
```mysql
SELECT DISTINCT
    < select_list >
FROM
    < left_table > < join_type >
JOIN < right_table > ON < join_condition >
WHERE
    < where_condition >
GROUP BY
    < group_by_list >
HAVING
    < having_condition >
ORDER BY
    < order_by_condition >
LIMIT < limit_number >
```

## 语句执行过程
![](https://tva1.sinaimg.cn/large/008eGmZEly1goevgg7j2vj30i10dp3zn.jpg)

### FROM连接
首先，对SELECT语句执行查询时，对`FROM`关键字两边的表执行连接，会形成[[笛卡尔积]]，这时候会产生一个`虚表VT1(virtual table)`。

### ON过滤
然后对FROM连接的结果进行ON筛选，创建VT2，把符合记录的条件存在VT2中。

### JOIN连接
如果是`OUTER JOIN(left join、right join)`，那么这一步就将添加外部行，如果是left join就把ON过滤条件的左表添加进来，如果是right join，就把右表添加进来，从而生成新的虚拟表 VT3。

### WHERE过滤
对上一步生产的虚拟表引用 WHERE 筛选，生成虚拟表 VT4。

WHERE 和 ON 的区别：
- 如果有外部列，ON针对过滤的是关联表，主表(保留表)会返回所有的列;
- 如果没有添加外部列，两者的效果是一样的;

应用：
- 对主表的过滤应该使用WHERE;
- 对于关联表，先条件查询后连接则用ON，先连接后条件查询则用WHERE;

### GROUP BY
根据group by字句中的列，会对VT4中的记录进行分组操作，产生虚拟机表VT5。如果应用了group by，那么后面的所有步骤都只能得到的VT5的列或者是聚合函数（count、sum、avg等）。

### HAVING
紧跟着GROUP BY字句后面的是HAVING，使用HAVING过滤，会把符合条件的放在VT6。

### SELECT
将VT6中的结果进行筛选，生成VT7.

### DISTINCT
对TV7生成的记录进行去重操作，生成VT8。事实上如果应用了group by子句那么distinct是多余的，原因同样在于，分组的时候是将列中唯一的值分成一组，同时只为每一组返回一行记录，那么所以的记录都将是不相同的。

### ORDER BY
应用order by子句。按照order_by_condition排序VT8，此时返回的一个游标，而不是虚拟表。sql是基于集合的理论的，集合不会预先对他的行排序，它只是成员的逻辑集合，成员的顺序是无关紧要的。

