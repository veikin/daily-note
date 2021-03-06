
### 为什么大多数数据库使用b-tree/b+tree而不是其他tree
1. 索引在磁盘，所以io次数是最重要指标
2. 磁盘存取原理（预读）
3. 　　B-Tree中一次检索最多需要h-1次I/O（根节点常驻内存），渐进复杂度为O（h）=O（logmN）。一般实际应用中，m是非常大的数字，通常超过100，因此h非常小（通常不超过3）。

　　综上所述，用B-Tree作为索引结构效率是非常高的。

　　而红黑树这种结构，h明显要深的多。由于逻辑上很近的节点（父子）物理上可能很远，无法利用局部性，所以红黑树的I/O渐进复杂度也为O（h），效率明显比B-Tree差很多
### B-Tree
- 每一个数据为kv形式，不同数据key不会重复。data为key以外的数据
- 一个节点中的key从左到右增加排列
- 查找数据时，从根结点二分查找。如果找到则返回。否则对应区间的指针节点递归进行查到。
- 插入删除新的数据会破坏b-tree的形式，需要对树进行分裂，合并，转移等操作。
### B+Tree
- 内节点不存储data，只存储key。叶子节点不存储指针。
![](https://i.loli.net/2018/12/04/5c067473a9daf.png)
- 由于并不是所有节点都具有相同的域，因此B+Tree中叶节点和内节点一般大小不同
- b+tree是b-tree基础上的一种优化。更适合外存储索引结构。
- B树中任何一个关键字只出现在一个结点中，而B+树中的关键字必须出现在叶节点中，也可能在非叶结点中重复出现
- 数据记录都存放在叶子节点中


### 数据库系统使用的b+tree
- 增加了顺序访问：提高区间访问性能

![](https://i.loli.net/2018/12/04/5c0676161dcde.png)
- 系统从磁盘读取数据到内存是以磁盘快为基本单位，位于同一个磁盘快的数据会被一次性读取出来
- InnoDB有页的概念。页是磁盘管理的最小单位，默认每页大小为16kb。可以通过show variables like 'innodb_page_size'查看
- InnoDB每次申请空间都是若干连续磁盘快达到页的大小16KB.
- 所有叶子节点之间是一种链式环结构
- B+Tree索引分为聚集索引和辅助索引
- IO读写次数是影响索引检索效率的最大因素！

总结一下：B+较于B更适合数据库索引的原因,关键点在于索引以索引文件存于磁盘中，减少了io次数
1. B+内部没有数据指针，容量更大。
2. 数据库中范围查询是非常频繁。B+叶子节点使用指针顺序链接。只要遍历叶子节点就可以实现整颗树的遍历。
3. 每一个关键字查询效率相当
4. 插入删除新的数据记录会破坏B-Tree的性质，因此在插入删除时，需要对树进行一个分裂、合并、转移等操作以保持B-Tree性质。造成IO操作频繁



### mysql的B-tree
####MYISAM
##### 主键索引
- 叶节点的data域存放的是数据记录的地址
- 数据文件与表文件分离
- 底层存储结构： frm -表定义、 myi -myisam索引、 myd-myisam数据
##### 辅助索引
- key可以重复
####INNODB
##### 主键索引
- 叶节点包含完整的数据记录，这种索引叫做聚集索引
- 数据文件本身按照主键聚集，所以必须要有主键
##### 辅助索引
- 所有辅助索引都引用主键作为data域
![](https://i.loli.net/2018/12/04/5c069f652fad3.png)
- 辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。



