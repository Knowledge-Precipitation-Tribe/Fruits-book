1. 首先将给定的url调用hash方法计算出对应的hash的value，在10亿的url中相同url必然有着相同的value。
2. 将文件的hash table放到第value%n台机器上。
3. value/n是机器上hash table的值。

分析：
将文件的url进行hash，得到值value，相同的url的文件具有相同的value，所以会被分配到同一台机器v%n上。在同一台机器上的重复的url文件具有相同的value/n值，如果出现了冲突，不同的url在同一台机器上也可能有相同的value/n值。在每个机器上将value/n值作为key，url值作为value构成hash表进行去重。最后将内存中去重后的hash表中的value值即url写入磁盘。合并磁盘中的各部分url文件，完成去重。