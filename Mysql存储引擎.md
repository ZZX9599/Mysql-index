# 索引

# 1:索引概述

## 1.1:介绍

索引是帮助MySQL高效获取数据的数据结构(该数据结构是有序的)，在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据， 这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引

本质：索引就是一种数据结构

作用：加速查找数据

## 1.2:无索引查找

演示

![image-20220901232605072]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220901232605072.png)

假如我们要执行的SQL语句为 ： select * from user where age = 45



![image-20220901232628117]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220901232628117.png)

无索引情况下，就需要从第一行开始扫描，直到最后一行，我们称之为全表扫描，性能很低

## 1.3:有索引演示

如果我们针对于这张表的age建立了索引，假设索引结构就是二叉树【实际上不是】，那么也就意味着，会对age这个字段建立一个二叉树的索引结构

![image-20220901232826220]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220901232826220.png)



假如我们要执行的SQL语句为 ： select * from user where age = 45

此时我们在进行查询时，只需要扫描三次就可以找到数据了，极大的提高的查询的效率

先查根节点36不符合-->45比36大-->右边-->48-->45比48小-->左边-->45【找到了】

这里我们只是假设索引的结构是二叉树，介绍一下索引的大概原理

只是一个示意图，并不是索引的真实结构



![image-20220901233255853]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220901233255853.png)

因为我们在insert update delete的时候，也需要对数据的修改进行索引数据结构的维护

# 2:索引结构



## 2.1:索引结构分类

索引是在引擎层实现的，对于不同的存储引擎，索引结构是不同的，主要有以下几种

![image-20220901233551687]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220901233551687.png)

## 2.2:索引支持情况

下图是MySQL中所支持的所有的索引结构，以下是不同的存储引擎对于索引结构的支持情况

![image-20220902001802962]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902001802962.png)

我们平常所说的索引，如果没有特别指明，都是指Innodb引擎下的B+树结构组织的索引

## 2.3:二叉树

一个节点下面最多包含两个子节点

二叉树特点是每个节点最多只能有两棵子树，且有左右之分

二叉树是n个有限元素的[集合](https://baike.baidu.com/item/集合/2908117?fromModule=lemma_inlink)，该集合或者为空

或者由一个称为根（root）的元素及两个不相交的、被分别称为左子树和右子树的二叉树组

成，是有序树

当集合为空时，称该二叉树为空二叉树

在二叉树中，一个元素也称作一个节点

![image-20221230203042632]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20221230203042632.png)

## 2.4:二叉排序树

二叉排序树（Binary Sort Tree），又称二叉查找树（Binary Search Tree）

亦称[二叉搜索树](https://baike.baidu.com/item/二叉搜索树/7077855?fromModule=lemma_inlink)，是[数据结构](https://baike.baidu.com/item/数据结构/1450?fromModule=lemma_inlink)中的一类。在一般情况下，查询效率比链表结构要高



特点：

- 左子树所有结点的值均小于根结点的值
- 右子树所有结点的值均大于根结点的值

![image-20220902091016332]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902091016332.png)



存在的问题：

如果主键是顺序插入的，则会形成一个单向链表，结构如下

![image-20220902091042068]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902091042068.png)

查找数字 17 就相当于遍历整个链表

所以，如果选择二叉树作为索引结构，会存在以下缺点： 

- 顺序插入时，会形成一个链表，查询性能大大降低。 
- 由于最多两个节点，大数据量情况下，层级较深，检索速度慢

我们可以选择平衡二叉树，那这样即使是顺序插入数据，最终形成的数据结构也是一颗平衡的二叉树

## 2.5:平衡二叉树

**平衡二叉树**又被称为AVL树



二叉平衡树有以下规则：

- 规则1：每个节点最多只有两个子节点（二叉）
- 规则2：每个节点的值比它的左子树所有的节点大
- 规则3：每个节点的值比它的右子树所有的节点小
- 规则4：每个节点左子树的高度与右子树高度之差的绝对值不超过1



相比于二叉排序树，就多了这个规则4



它是一棵空树或它的左右两个子树的高度差的绝对值不超过1

并且左右两个子树都是一棵平衡[二叉树](https://so.csdn.net/so/search?q=二叉树&spm=1001.2101.3001.7020)



这个方案很好的解决了二叉排序树退化成链表的问题

但是频繁旋转会使插入和删除牺牲掉O(logN)左右的时间

不过相对二叉排序树来说，时间上稳定了很多

![image-20221230203643886]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20221230203643886.png)

这样层级就不会那么深了

但是频繁的插入或者删除或者更新数据，则容易造成不满足左右最大高度差不为1

则又需要重新构建这个树，就会重新旋转来保持平衡，而旋转非常耗时的

所以平衡二叉树（AVL）适合用于插入与删除次数比较少，但查找多的情况

缺点还是一样的：因为最大只有两个节点，大数据量的时候还是层级较深

## 2.6:红黑树

红黑树的规则：

- 规则1：每个节点不是黑色就是红色
- 规则2：根节点为黑色
- 规则3：红色节点的父节点和子节点不能为红色
- 规则4：所有的叶子节点都是黑色（空节点视为叶子节点NIL）
- 规则5：每个节点到叶子节点的每个路径黑色节点的个数都相等



平衡二叉树和红黑树的区别

- 平衡二叉树的左右子树的高度差绝对值不超过1，但是红黑树在某些时刻可能会超过1，只要符合红黑树的五个条件即可
- 二叉树只要不平衡就会进行旋转，而红黑树不符合规则时，有些情况只用改变颜色不用旋转，就能达到平衡
- 平衡二叉树是严格的平衡二叉树，平衡条件必须满足（所有节点的左右子树高度差不超过1）不管我们是执行插入还是删除操作，只要不满足上面的条件，就要通过旋转来保持平衡，而旋转非常耗时的。所以平衡二叉树（AVL）适合用于插入与删除次数比较少，但查找多的情况
- 红黑树在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O(logn)。所以红黑树适用于搜索，插入，删除操作较多的情况

![image-20220902091617704]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902091617704.png)



优点：解决了平衡二叉树针对数据修改的时候需要经常进行旋转平衡的操作

缺点：大数据量情况下，层级较深，检索速度慢【二叉树都存在的问题】



正是因为这个缺点【大数据量的时候层级过深，而一般项目的数据量都很大】

所以，在MySQL的索引结构中，并没有选择二叉树或者红黑树

而选择的是B+Tree，那么什么是 B+Tree呢？在详解B+Tree之前，先来介绍一个B-Tree



## 2.7:B-Tree

B-Tree：又叫做B树，是一种多叉路衡查找树，相对于二叉树，B树每个节点可以有多个分支，即多叉。以一颗最大度数(max-degree)为5(5阶)的b-tree为例，那这个B树每个节点最多存储4个key，5 个指针：

最大度数:每个节点的最多子节点个数

![image-20220902092937797]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902092937797.png)

可视化网站：https://www.cs.usfca.edu/~galles/visualization/BTree.html



构建过程：度为5



先加入四个元素

![image-20220902093824248]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902093824248.png)

再加入一个15，成为 12,15,35,45,98

中间元素向上分裂，形成两侧

![image-20220902093910140]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902093910140.png)



再加入100，比35大，走右边

![image-20220902094020984]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094020984.png)



再加入199，比35大，走右边

![image-20220902094049961]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094049961.png)



再加入88，比35大，走右边，但是超过了五阶，中间元素98向上分裂

![image-20220902094154048]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094154048.png)



再插入23，26

![image-20220902094326442]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094326442.png)



再插入32

![image-20220902094344002]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094344002.png)

特点： 

- 5阶的B树，每一个节点最多存储4个key，对应5个指针
- 一旦节点存储的key数量到达5，就会裂变，中间元素向上分裂
- 在B树中，非叶子节点和叶子节点都会存放数据【注意和B+树区别】
- 叶子节点和非叶子节点都会保存数据，导致页的存储数据量变小



## 2.7:B+树

B+Tree是B-Tree的变种，我们以一颗最大度数（max-degree）为4（4阶）的b+tree为例，来看一 下其结构示意图

![image-20220902094704107]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094704107.png)

**可视化：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html**



最终我们看到，B+Tree 与 B-Tree相比，主要有以下三点区别： 

- 所有的数据都会出现在叶子节点
- 叶子节点形成一个单向链表 
- 非叶子节点仅仅起到索引数据作用，具体的数据都是在叶子节点存放的



演示：

先添加四个元素：

![image-20220902094811235]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902094811235.png)



再添加一个 176，则234是中间元素，中间元素向上分裂的同时，会把自己添加到叶子节点并形成单向链表

![image-20220902095107188]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902095107188.png)

再添加一个元素654

![image-20220902095148274]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902095148274.png)



再添加一个元素666

![image-20220902095218910]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902095218910.png)

上述我们所看到的结构是标准的B+Tree的数据结构【叶子结点的数据构成单向链表】

接下来，我们再来看看MySQL中优化之后的 B+Tree



## 2.8:Mysql优化的B+树

MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点 的链表指针构成了双向链表，就形成了带有顺序指针的B+Tree，提高区间访问的性能，利于排序

![image-20220902095531710]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902095531710.png)

上面的非叶子节点只是起到索引的作用，叶子节点才是真实存储数据的

页是组成区的最小单元，页也是InnoDB 存储引擎磁盘管理的最小单元

每个页的大小默认为 16KB



## 2.9:Hash

MySQL中除了支持B+Tree索引，还支持一种索引类型---Hash索引。

结构：哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位

上，然后存储在 hash表中

![image-20220902095808802]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902095808802.png)

![image-20220902095827603]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902095827603.png)

特点 

- Hash索引只能用于对等比较(=，in)，不支持范围查询（between，>，< ，...） 
- 无法利用索引完成排序操作 
- 查询效率高(如果不存在hash冲突的情况)只需要一次检索就可以了，效率通常要高于B+tree索引



存储引擎支持

在MySQL中，支持hash索引的是Memory存储引擎

而InnoDB中具有自适应hash功能

hash索引是 InnoDB存储引擎根据B+Tree索引在指定条件下自动构建的

## 2.10:思考题

思考题:为什么InnoDB存储引擎选择使用B+tree索引结构

本道题其实就是为了对比以上常见的数据结构的区别

- 相对于二叉树，层级更少，搜索效率高
- 对于B-tree【B树】，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低 
- 相对Hash索引，B+tree支持范围匹配及排序操作
- 还可以说出mysql的B+树进行了优化，改为了双向的链表指针



# 3:索引分类

## 3.1:索引常见分类

在MySQL数据库，将索引的具体类型主要分为以下几类：

主键索引、唯一索引、常规索引、全文索引

![image-20220902100509939]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902100509939.png)

事实上，加入主键或者唯一约束等信息之后，就会默认创建索引了



## 3.2:聚集索引&二级索引

有的称作为：聚簇索引与非聚簇索引

在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种：

![image-20220902100637789]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902100637789.png)



聚集索引的索引结构下面，把整行的数据关联到了一起

非聚集索引【二级索引】的索引结构下面，关联的是对应数据的主键



没有指定索引的话，存在聚集索引自动选取规则：

- 如果存在主键，主键索引就是聚集索引
- 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引
- 如果表没有主键，并且没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引

也就是无论如何，聚集索引都会存在



聚集索引结构：

下面的假设ID为主键，则默认为ID创建了主键索引，则ID这个就是聚集索引

![image-20220902101052954]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902101052954.png)

聚集索引的叶子节点的数据就是这一行的数据

比如叶子节点5下面悬挂的row就是  `id:5, name:kit, gender:男`



由于聚集索引只存在一个，所以name的索引就不是聚集索引，而是二级索引

![image-20220902101228654]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902101228654.png)

二级索引下面的数据存放的是该字段值对应的主键值

例如Arm 下面悬挂的数据是 id=10



聚集索引的叶子节点下挂的是这一行的数据 

二级索引的叶子节点下挂的是该字段值对应的主键值



## 3.3:分析执行SQL语句

接下来，我们来分析一下，当我们执行如下的SQL语句时，具体的查找过程是什么样子的

![image-20220902101449038]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902101449038.png)



```apl
where name='Arm'
不会走聚集索引，走了name的二级索引
二级索引叶子节点悬挂的是当前数据的id
找到对应的id

但是由于我们是select *，就会进行回表查询
根据id查询数据，走聚集索引
聚集索引下面的是这一行的数据
就查到了对应的行数据
```



回表查询： 这种先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取数据的方式，就称之为回表查询



## 3.4:思考题

以下两条SQL语句，那个执行效率高? 为什么? 

- select * from user where id = 10 
- select * from user where name = 'Arm'
- 备注: id为主键，name字段创建的有索引



```apl
因为id为主键，第一条根据id查询，走聚集索引，聚集索引的数据结构的叶子节点悬挂的就是当前行数据，则直接就返回该数据了，不需要再进行任何额外的查询列

第二条，name属于二级索引，叶子节点下面悬挂的是当前的id会进行回表查询，效率更低
```



InnoDB主键索引的B+tree高度为多高呢?

![image-20220902102536581]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902102536581.png)

每个节点都是一个页

每个页的大小默认为 16 KB

所以每个页能存储的key和指针是有限的



假设: 

一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB的指针占用6个字节的空间【固定】

主键为int，四个字节，即使为bigint，占用字节数为8



如果高度为2

n * 8 + (n + 1) * 6 = 16*1024 , 算出n约为 1170

```apl
n*8   ====> 主键为bigint的占用字节
(n+1)*6  ===> 指针所占用的字节
16*1024 ===> 每页16k固定的，也就是16*1024字节
算出n约为 1170
1171* 16 = 18736
```



如果高度为3

1171 * 1171 * 16 = 21939856 

也就是说，如果树的高度为3，则可以存储 2200w 左右的记录



# 4:索引语法

## 4.1:数据准备

```sql
create table tb_user(
	id int primary key auto_increment comment '主键',
	name varchar(50) not null comment '用户名',
	phone varchar(11) not null comment '手机号',
	email varchar(100) comment '邮箱',
	profession varchar(11) comment '专业',
	age tinyint unsigned comment '年龄',
	gender char(1) comment '性别 , 1: 男, 2: 女',
	status char(1) comment '状态',
	createtime datetime comment '创建时间'
) comment '系统用户表';


INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('吕布', '17799990000', 'lvbu666@163.com', '软件工程', 23, '1', '6', '2001-02-02 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('曹操', '17799990001', 'caocao666@qq.com', '通讯工程', 33, '1', '0', '2001-03-05 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('赵云', '17799990002', '17799990@139.com', '英语', 34, '1', '2', '2002-03-02 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('孙悟空', '17799990003', '17799990@sina.com', '工程造价', 54, '1', '0', '2001-07-02 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('花木兰', '17799990004', '19980729@sina.com', '软件工程', 23, '2', '1', '2001-04-22 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('大乔', '17799990005', 'daqiao666@sina.com', '舞蹈', 22, '2', '0', '2001-02-07 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('露娜', '17799990006', 'luna_love@sina.com', '应用数学', 24, '2', '0', '2001-02-08 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('程咬金', '17799990007', 'chengyaojin@163.com', '化工', 38, '1', '5', '2001-05-23 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('项羽', '17799990008', 'xiaoyu666@qq.com', '金属材料', 43, '1', '0', '2001-09-18 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('白起', '17799990009', 'baiqi666@sina.com', '机械工程及其自动化', 27, '1', '2', '2001-08-16 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('韩信', '17799990010', 'hanxin520@163.com', '无机非金属材料工程', 27, '1', '0', '2001-06-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('荆轲', '17799990011', 'jingke123@163.com', '会计', 29, '1', '0', '2001-05-11 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('兰陵王', '17799990012', 'lanlinwang666@126.com', '工程造价', 44, '1', '1', '2001-04-09 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('狂铁', '17799990013', 'kuangtie@sina.com', '应用数学', 43, '1', '2', '2001-04-10 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('貂蝉', '17799990014', '84958948374@qq.com', '软件工程', 40, '2', '3', '2001-02-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('妲己', '17799990015', '2783238293@qq.com', '软件工程', 31, '2', '0', '2001-01-30 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('芈月', '17799990016', 'xiaomin2001@sina.com', '工业经济', 35, '2', '0', '2000-05-03 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('嬴政', '17799990017', '8839434342@qq.com', '化工', 38, '1', '1', '2001-08-08 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('狄仁杰', '17799990018', 'jujiamlm8166@163.com', '国际贸易', 30, '1', '0', '2007-03-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('安琪拉', '17799990019', 'jdodm1h@126.com', '城市规划', 51, '2', '0', '2001-08-15 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('典韦', '17799990020', 'ycaunanjian@163.com', '城市规划', 52, '1', '2', '2000-04-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('廉颇', '17799990021', 'lianpo321@126.com', '土木工程', 19, '1', '3', '2002-07-18 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('后羿', '17799990022', 'altycj2000@139.com', '城市园林', 20, '1', '0', '2002-03-10 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('姜子牙', '17799990023', '37483844@qq.com', '工程造价', 29, '1', '4', '2003-05-26 00:00:00');
```



## 4.2:创建索引

```sql
CREATE [ UNIQUE | FULLTEXT ] INDEX index_name ON table_name (index_col_name,..)
```

**创建唯一索引，全文索引，不写就是常规索引**

索引的规范命名：`index_表名_字段1_字段2....`

或者是`idx_表名_字段1_字段2....`



## 4.3:查看索引

```sql
SHOW INDEX FROM table_name
SHOW INDEX FROM table_name\G  转换为列
```



## 4.4:删除索引

```sql
DROP INDEX index_name ON table_name 
```



## 4.5:语法练习

先查看索引，发现只有主键索引

![image-20220902104533868]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902104533868.png)



需求：

user表的name字段为姓名字段，该字段的值可能会重复，为该字段创建索引

```sql
create index idx_user_name on tb_user(name)
```

再次查看索引信息

![image-20220902104757721]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902104757721.png)

phone手机号字段的值，是非空，且唯一的，为该字段创建唯一索引

```sql
create unique index idx_user_phone on tb_user(phone);
```

![image-20220902104937153]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902104937153.png)

为profession、age、status创建联合索引。

```sql
create index idx_user_pro_age_sta on tb_user(profession,age,status);
```

![image-20220902105120972]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902105120972.png)

为email建立合适的索引来提升查询效率。

```sql
create index idx_user_email on tb_user(email);
```

![image-20220902105216200]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902105216200.png)

# 5:性能分析



## 5.1:SQL执行频率(了解)

MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信 息。通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次：

知道了执行频率，我们才知道怎么在线上对哪些sql来进行优化

```sql
-- session 是查看当前会话 ;
-- global 是查询全局数据 ;
SHOW GLOBAL STATUS LIKE 'Com_______';  一个_代表匹配一个字符
```



![]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902105546373.png)



我们执行一次select * from tb_user

再执行SHOW GLOBAL STATUS LIKE 'Com_______'

发现数量增加了1



那么通过查询SQL的执行频次，我们就能够知道当前数据库到底是增删改为主，还是查询为主

那假 如说是以查询为主，我们又该如何定位针对于那些查询语句进行优化呢？ 

我们可以借助于慢查询日志



## 5.2:慢查询日志(了解)

```sql
先导入一千万的数据到数据库
一次性需要插入大批量数据，使用insert语句插入性能较低
此时可以使用MysQL数据库提供的load指令进行插入。
准备1000W的数据在tb_sku.sql下

连接数据库的时候：mysql --local-infile -u root -p

load data local infile '/root/sql/tb_sku.sql' into table `tb_sku` fields terminated by ',' lines terminated by '\n';
```

慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有 SQL语句的日志。 

MySQL的慢查询日志默认没有开启，我们可以查看一下系统变量 slow_query_log。

```sql
#查看是否开启慢查询日志，OFF代表每页，默认为OFF
show variables like 'slow_query_log'  
```

如果要开启慢查询日志，需要在Linux的MySQL的配置文件（/etc/my.cnf）中配置如下信息：

```apl
在[Mysqld]下加入
# 开启MySQL慢日志查询开关
slow_query_log=1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
# 慢查询日志文件位置
slow_query_log_file = /var/log/mysqld-slow.log
```



在/var/log/下使用  tail -f mysqld-slow.log可以查看文件的追加情况



执行 select count(*) from tb_sku 明显大于2S

这边输出了慢查询日志 用户，ip，花费时间，执行的语句等

![image-20220902154344483]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902154344483.png)



最终我们发现，在慢查询日志中，只会记录执行时间超多我们预设时间（2s）的SQL，执行较快的SQL 是不会记录的

那这样，通过慢查询日志，就可以定位出执行效率比较低的SQL，从而有针对性的进行优化。



## 5.3:profile详情(了解)

show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了

通过have_profiling 参数，能够看到当前MySQL是否支持profile操作

```sql
SELECT @@have_profiling
```

返回Yes即代表支持，支持但不是代表开启了

```sql
SELECT @@profiling
```

返回0，代表没有开启

可以通过set语句在 session/global级别开启profiling

```sql
SET profiling = 1
```

开关已经打开了，接下来，我们所执行的SQL语句，都会被MySQL记录，并记录执行时间消耗到哪儿去 了。 我们直接执行如下的SQL语句

```sql
select * from tb_user;
select * from tb_user where id = 1;
select * from tb_user where name = '白起';
select count(*) from tb_sku;
```



执行后执行 show profiles

![image-20220902155017002]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902155017002.png)



假如我想知道select count(*) from tb_sku耗费时间到哪里去了

我们就可以执行一个命令查看当前SQL时间都耗费到哪里去了

查看指定query_id的SQL语句各个阶段的耗时情况
show profile for query query_id;



以上的query_id也就是5

```sql
show profile for query 5;
```

![image-20220902155347911]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902155347911.png)

我们发现耗费时间主要是在executing【执行】



查看指定query_id的SQL语句CPU的使用情况 

show profile cpu for query query_id;

```sql
show profile cpu for query 5;
```

![image-20220902155529427]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902155529427.png)



## 5.4:explain执行计划

上面的时间只是初略判断，对于开发人员来讲，了解即可，这个执行计划对开发人员是比较重要的

EXPLAIN 或者 DESC命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行 过程中表如何连接和连接的顺序

```sql
-- 直接在select语句之前加上关键字 explain / desc【描述】
EXPLAIN SELECT 字段列表 FROM 表名 WHERE 条件 ;
```

例如 explain select * from tb_user where id=1

![image-20220902155808862]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902155808862.png)

Explain 执行计划中各个字段的含义：

### 1:id

select 查询的序列号，表示查询中执行 select 子句或者操作表的顺序（id相同，执行顺序从上到下；id不同，值越大越先执行）多表查询的时候才有用

这里使用一个多表联查

![image-20220902160817232]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902160817232.png)

id相同的话，表结构的执行顺序就是从上到下

嵌套子查询的时候，最内层的表的id最大，最内层最先执行



### 2:select_type

表示 SELECT 的类型，常见取值有 SIMPLE（简单表，即不适用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION中的第二个或者后面的查询语句）、SUBQUERY（SELECT/WHERE之后包含了子查询）等



### 3:type

表示连接类型，性能由好到差的连接类型为 NULL、system、const、eq_ref、ref、range、index、all，不可能优化到null，除非是 类似 select '10' 直接输出的

访问系统表，可能会出现system

根据主键，唯一索引的，就会出现const

使用非唯一性的索引，会出现 ref 

出现all，代表全表扫面

出现index，代表使用了索引，但是还是做了全表扫描



### 4:possible_key(重点)

可能应用在这张表上的索引，一个或多个



### 5:key(重点)

实际使用的索引，如果为 NULL，则没有使用索引



### 6:key_len(重点)

表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好



### 7:rows

MySQL认为必须要执行的行数，在InnoDB引擎的表中，是一个估计值，可能并不总是准确的



### 8:filtered(重点)

表示返回结果的行数占需读取行数的百分比，filtered的值越大越好



#### 3.5.4.9:我们最关注的是

- possible_key
- key
- key_len
- filtered

# 6:索引使用

## 6.1:验证索引效率

我们还是使用之前准备的一张表 tb_sku , 在这张表中准备了1000w 的记录

![image-20220902163421389]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902163421389.png)



我们在建表的时候添加了主键，就默认添加了主键索引，根据id主键查询，注意观察执行时间

![image-20220902163521135]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902163521135.png)

根据sn查询，我们暂未设置索引，也就是不走索引，查看时间

![image-20220902163647065]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902163647065.png)

验证：我们给sn创建一个索引

create index idx_sku_sn on tb_sku(sn)



注意：由于我们的数据量太大，创建索引的时间也会很长，原因是因为要对数据创建B+树的数据结构来维护数据，所以耗时也是较长的



这个时候，我们在执行一次刚刚的根据sn查询的sql语句

![image-20220902164216593]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902164216593.png)

发现时间大大的缩短，即验证了索引对于效率的提升



查看执行计划，有没有使用索引

![image-20220902164337468]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902164337468.png)

## 6.2:最左前缀法则

如果索引了多列索引【把多个列联合起来作为了联合索引】，这个索引就需要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，索引将会部分失效【后面的字段索引失效】



添加联合索引  idx_user_pro_age_sta

![image-20220902164556641]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902164556641.png)



意思是：要让这个联合索引生效的话，则必须保证最左边存在【profession】

中间跳过【age】的话，则后面的索引【status】都会失效



```sql
explain select * from tb_user 
where profession = '软件工程' and age = 31 and status= '0';
```



分析：

是从最左侧profession开始【整体不失效】

并且没跳过中间任何的联合索引字段【不存在联合索引局部失效】



执行计划使用的索引信息

![image-20220902164957283]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902164957283.png)



```sql
explain select * from tb_user 
where profession = '软件工程' and age = 31;
```



分析：

是从最左侧profession开始【整体不失效】

并且没跳过中间任何的联合索引字段【不存在联合索引局部失效】



执行计划使用的索引信息

![image-20220902165058647]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902165058647.png)

注意到key_len少了4，证明status的索引字段长度为4

```sql
explain select * from tb_user where profession = '软件工程';
```



分析：

是从最左侧profession开始【整体不失效】

并且没跳过中间任何的联合索引字段【不存在联合索引局部失效】



执行计划使用的索引信息

![image-20220902165336358]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902165336358.png)

注意到key_len少了2，证明age的索引字段长度为2



得到索引长度  

- profession=36 
- age=2 
- status=4



```sql
explain select * from tb_user where age = 31 and status = '0';
explain select * from tb_user where status = '0';
```

分析：不是从最左侧profession开始的【则索引整体失效】

执行计划使用的索引信息

![image-20220902165522253]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902165522253.png)





```sql
explain select * from tb_user 
where profession = '软件工程' and status = '0';
```

分析：

是从最左侧profession开始【整体不失效】

但是跳过了中间的联合索引字段age【联合索引局部失效，从age包括之后的失效】

![image-20220902165726777]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902165726777.png)



思考:

当执行SQL语句: 

```sql
explain select * from tb_user 
where age = 31 and status = '0' and profession = '软件工程' 
```

是否满足最左前缀法则，走不走上述的联合索引，索引长度？



```apl
会走索引，实际上他并不是要求你的profession必须放在第一个，他的意思是存在即可
```



## 6.3:联合索引范围查询失效

在联合索引中，出现范围查询(   >(大于)或者<(小于)  )，会导致范围查询右侧的列索引失效

```sql
explain select * from tb_user 
where profession = '软件工程' and age > 30 and status = '0';
```



![image-20220902170200666]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902170200666.png)

key_len的长度为38，说明只有profession和age走了索引，status索引失效

注意：age并未失效，age之后的才失效了



解决方式：

在业务允许的情况下尽量使用 >= 和 <=进行范围查询

```sql
explain select * from tb_user 
where profession = '软件工程' and age >= 30 and status = '0';
```

![image-20220902170403226]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902170403226.png)



## 6.4:索引失效情况

### 1:索引列运算(运)

不要在索引列上进行运算操作， 索引将失效。

我们的表tb_user存在一个索引 idx_user_phone

![image-20220902170642422]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902170642422.png)

当根据phone字段进行函数运算操作之后，索引失效

```sql
# 查询电话号码结尾两位数为15的信息
explain select * from tb_user where substring(phone,10,2) = '15';
```



查看执行计划，发现不走索引

![image-20220902170844253]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902170844253.png)



### 2:字符串不加引号(型)

接下来，我们通过两组示例，来看看对于字符串类型的字段，加单引号与不加单引号的区别



phone字段为varchar类型，并且已经对phone字段添加了索引

```sql
explain select * from tb_user where phone = '17799990015';
explain select * from tb_user where phone = 17799990015;
```



字符串类型加上了引号

![image-20220902171125416]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902171125416.png)



不加引号

![image-20220902171200930]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902171200930.png)



经过上面两组示例，我们会明显的发现，如果字符串不加单引号，对于查询结果，没什么影响，但是数据库存在隐式类型转换，索引将失效



### 3:模糊查询(模)

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

接下来，我们来看一下这三条SQL语句的执行效果，查看一下其执行计划： 

由于下面查询语句中，都是根据profession字段查询，符合最左前缀法则

联合索引整体是可以生效的

我们主要看一下，模糊查询时，%加在关键字之前，和加在关键字之后的影响

```sql
explain select * from tb_user where profession like '软件%';
explain select * from tb_user where profession like '%工程';
explain select * from tb_user where profession like '%工%';
```



第一条走了索引

![image-20220902171805041]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902171805041.png)



第二条不走

![image-20220902171859268]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902171859268.png)





第三条不走

![image-20220902171909756]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902171909756.png)



经过上述的测试，我们发现，在like模糊查询中，在关键字后面加%，索引可以生效

而如果在关键字 前面加了%，索引将会失效



### 4:or连接条件

用or分割开的条件， 如果or前的条件中的列有索引，而后面的列中没有索引，或者or前的没有，or后的有，那么涉及的索引都不会被用到，只有两边都有的，才走索引

```sql
#id有索引，age只存在联合索引，没使用profession，不符合最左前缀，所以age相当于没有索引
explain select * from tb_user where id = 10 or age = 23;
explain select * from tb_user where phone = '17799990017' or age = 23;
```



第一条执行计划：

![image-20220902172253891]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902172253891.png)



第二条执行计划：

![image-20220902172349381]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902172349381.png)



由于age没有索引，所以即使id、phone有索引

索引也会失效。所以需要针对于age也要建立索引。

```sql
create index idx_user_age on tb_user(age);
```



这一下再执行上面的sql，查看执行计划，发现索引成功了



### 5:数据分布影响

如果MySQL评估使用索引比全表更慢，则不使用索引

phone现在存在索引，phone的最小值目前是17799990000，最大是17799990023

```sql
explain select * from tb_user where phone >= '17799990023';
explain select * from tb_user where phone >= '17799990000';
```



第一条走了索引：

![image-20220902172943023]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902172943023.png)



第二条没走索引：

![image-20220902173023388]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902173023388.png)



MySQL在查询时，会评估使用索引的效率与走全表扫描的效率，如果走全表扫描更快，则放弃 索引，走全表扫描。 因为索引是用来索引少量数据的，如果通过索引查询返回大批量的数据，则还不如走全表扫描来的快，此时索引就会失效



我的表数据全部都是有数据的，不存在profession为Null的数据



同理：

```sql
# 这个查询结果一条数据都没有，走索引更快，所以mysql会评估，最终走索引
explain select * from tb_user where profession is null;

# 这个查询结果是全部的数据，也就是全表，所以mysql会评估，最终不走索引，直接全表扫描
explain select * from tb_user where profession is not null;
```

注意：is null 和 is not null和走不走索引跟关键字没关系，凭数据的分布决定，由mysql决定



### 6:小结

根据抖音的小结：模型数空运最快，拆解



模：模糊查询【使用like关键字，并且%在前，则索引失效】

型：数据类型【也就是数据类型错误了，索引失效，比如varcher的类型存入了数字，我们使用数字也可以查询出对应的结果，但是不会走索引】

数：函数【对索引的字段使用了内部的函数，索引失效】

空：Null的意思【索引不存储空值，如果索引列没有限制为Not Null，也就是数据可以为null的话，则数据库会认为索引列可能存在空值，也不会使用索引】

运：运算【对索引列进行加减乘除之类的运算，会失效】

最：最左前缀法则【联合索引需要满足最左前缀法则】

快：不走索引更快【如果MySQL评估使用索引比全表更慢，则不使用索引】





## 6.5:为SQL添加提示(了解)

目前tb_user只存在这下面几个索引

![image-20220902210311246]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902210311246.png)

执行  explain select * from tb_user where profession = '软件工程'   会走索引



我们假如再添加一个索引

```sql
create index idx_user_profession on tb_user(profession);
```



这个时候查看sql执行计划

![image-20220902210539399]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902210539399.png)

存在多个索引的时候，可能用到的索引有两个

但是最后用到的是联合索引，这个是Mysql自己评判的



那么，我们能不能在查询的时候，自己来指定使用哪个索引呢？ 

答案是肯定的，此时就可以借助于 MySQL的SQL提示来完成

接下来，介绍一下SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说

就是在SQL语句中加入一些人为的提示来达到优化操作的目的



- use index：建议MySQL使用哪一个索引完成查询(仅仅是建议，内部还会再次进行评估)
- ignore index：略指定的索引
- force index：强制使用索引



我们假如想使用我们刚刚创建的单列索引，就可以加上提示

```sql
explain select * from tb_user 
use index(idx_user_profession) 
where profession = '软件工程';
```

![image-20220902211030600]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902211030600.png)



忽略单列索引

```sql
explain select * from tb_user 
ignore index(idx_user_profession) 
where profession = '软件工程';
```

![image-20220902211116171]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902211116171.png)



强制使用

```sql
explain select * from tb_user 
force index(idx_user_profession) 
where profession = '软件工程';
```

![image-20220902211348612]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902211348612.png)



## 6.6:覆盖索引

尽量使用覆盖索引，减少 select *

那么什么是覆盖索引呢？ 覆盖索引是指查询使用了索引，并且需要返回的列，在该索引中

已经全部能够找到，关注的不是where之后的，而是select的字段都能在索引找到，这类查

询就叫做覆盖索引

接下来，我们来看一组SQL的执行计划，看看执行计划的差别，然后再来具体做一个解析



目前存在的索引是 profession,age, status



```sql
explain select id, profession from tb_user 
where profession = '软件工程' and age =31 and status = '0' ;


explain select id,profession,age, status from tb_user 
where profession = '软件工程'and age = 31 and status = '0' ;


explain select id,profession,age, status, name from tb_user 
where profession = '软件工程' and age = 31 and status = '0' ;


explain select * from tb_user 
where profession = '软件工程' and age = 31 and status= '0';
```



从上述的执行计划我们可以看到，这四条SQL语句的执行计划前面所有的指标都是一样的，看不出来差异。但是此时，我们主要关注的是后面的Extra，前面两条SQL的结果为 Using where; Using Index ; 而后面两条SQL的结果为: Null或Using index condition



Using where & Using index

查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据



Null & Using index condition

查找使用了索引，但是需要回表查询数据



```apl
在tb_user表中有一个联合索引 idx_user_pro_age_sta，该索引关联了三个字段
profession、age、status，而这个联合索引也是一个二级索引，因为我们这个表存在主键
所以聚集索引是PRIMARY，所以叶子节点下面挂的是这一行的主键id
所以当我们查询返回的数据在 id、profession、age、status 之中，则直接走二级索引
直接返回数据了。 如果超出这个范围，就需要拿到主键id，再去扫描聚集索引，再获取额外的数据了。这个过程就是回表。 而我们如果一直使用select * 查询返回所有字段值，很容易就会造成回表查询（除非是根据主键查询，此时只会扫描聚集索引）
```



我们一起再来看下面的这组SQL的执行过程

![image-20220902213612918]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902213612918.png)

- id是主键，是一个聚集索引
- name字段建立了普通索引，是一个二级索引（辅助索引）

![image-20220902213648294]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902213648294.png)

- 聚集索引挂的是这一行数据

- 二级索引节点挂的是主键id

  

执行SQL : select * from tb_user where id = 2;

直接走聚集索引直接拿到行数据就可以了



执行SQL：selet id,name from tb_user where name = 'Arm';

先走二级索引，拿到Arm和Arm下悬挂的id，不涉及回表

涉及的字段在这个索引的结构下都能找到，这个就叫做覆盖索引



执行SQL：selet id,name,gender from tb_user where name = 'Arm';

![image-20220902214046544]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902214046544.png)



发现在这个索引的数据结构只能找到部分，还差一个gender字段，这个就不是覆盖索引

还需要拿到节点下面的id，再进行回表查询

所以为什么不建议select *

正是因为根据除聚集索引以外的索引或者是全部字段的联合索引，都会涉及到回表查询



思考题：

一张表, 有四个字段(id, username, password, status) 由于数据量大

需要对以下SQL语句进行优化, 该如何进行才是最优方案: 

select id,username,password from tb_user where username = 'zhangsan';



问题解决：

- 性能优化，首先是加索引
- 查询的字段较多，容易回表查询，所以，我们可以针对username,password建立联合索引
- 这样就不会执行回表查询，直接执行了覆盖索引



## 6.7:前缀索引(了解)

当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串

这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率

此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引

效率，但是也可能存在误判的现象，如果取得太短则误判可能性更大，取的太长浪费磁盘



语法：

```sql
create index idx_xxxx on table_name(column(n)) ;
```

n代表索引的字符串长度

事实上，我们使用一个很大的数据的前缀就可以很好地拿到这个数据



前缀长度选择：

可以根据索引的选择性来决定，选择性是指不重复的索引值(基数)和数据表的记录总数的比值， 索引选择性越高则查询效率越高，唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的



假设tb_user的email是一个特别大的字段，维护数据结构消耗巨大，我们就可以这样来确定长度

两个参数：

表的总记录数

```sql
select count(*) from tb_user
```

不重复的记录数

```sql
select count(distinct email) from tb_user
```

计算

```sql
select count(distinct email) / count(*) from tb_user
```

当然我们不需要全部，只需要截取部分【假设截取前10个数据作为索引】

```sql
select count(distinct substring(email,1,10)) / count(*) from tb_user
```

假设截取前八个还是为1，我们还可以继续截取前九个

```sql
select count(distinct substring(email,1,10)) / count(*) from tb_user
```

我们的目的就是尽可能的让索引的选择性为1

我们最后加入选择了5个作为前缀



优点：减少了大数据量的建立索引或者维护索引的成本

缺点：如果在索引的选择性小于一的时候，回表的概率会增加



举例：

email为 536509593@qq.com 和 536509673@qq.com的两个邮箱

我们假如建立的索引只取了前5个字段，则我们查询的索引都是 53650

所以还是需要拿到数据下面的主键id，进行回表查询



前缀索引的查询流程

![image-20220902221603964]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902221603964.png)



## 6.8:单列索引与联合索引

- 单列索引：即一个索引只包含单个列 

- 联合索引：即一个索引包含了多个列



我们的phone和name都是有索引的，存在单列的phone和name索引，不存在联合索引

```sql
explain select * from tb_user where phone='17799990010' and name='韩信'
```



查看执行计划

![image-20220902231301604]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902231301604.png)

发现只走了phone的索引，没有走name的索引，phone的所有并不包含name

所以虽然加入了两个索引，但是两个索引就算走完了也不满足 select * 中的所有字段

也就是不满足覆盖索引，就算两个索引走完了，还是会执行回表索引，这样就消耗了时间

所以在执行SQL的时候，即使存在多个索引，Mysql会自己判定，如果执行的时候使用了多

个索引，一样不满足覆盖索引的话，就不会执行那么多的索引，因为这样反而降低了性能



通过上述执行计划我们可以看出来，在and连接的两个字段 phone、name上都是有单列索引的，但是最终mysql只会选择一个索引，也就是说，and连接的两个索引，只能走一个字段的索引，此时是会回表查询的



我们再来创建一个phone和name字段的联合索引来查询一下执行计划。

```sql
create unique index idx_user_phone_name on tb_user(phone,name);
```



执行sql语句

```sql
explain select * from tb_user where phone='17799990010' and name='韩信';
```

![image-20220902231620595]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902231620595.png)

发现三个索引，只执行了一个索引，走了联合索引，是mysql自己选择的，满足覆盖索引



在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。因为涉及到多个and的单列索引，但是mysql最后还是只执行了一个索引，这样就不满足了覆盖索引，最后还是会走回表查询



如果查询使用的是联合索引，联合索引的具体结构示意图如下：

![image-20220902232119422]( https://zzx-note.oss-cn-beijing.aliyuncs.com/mysql/image-20220902232119422.png)

会先进行phone排序，phone相同的时候再根据name排序

同样属于二级索引，悬挂的依然是主键ID

当根据联合索引查询联合索引的字段的时候，就满足覆盖查询，不会进行回表查询

建立联合字段的时候，字段的顺序是有讲究的，而且还有 最左前缀法则 等约束



## 6.9:索引设计原则

针对于数据量较大，且查询比较频繁的表建立索引【超过百万一般就算比较大了】

针对于常作为查询条件(where)排序(order by)分组(group by)操作的字段建立索引

尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高

如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引

尽量使用联合索引，减少单列索引，查询字段较多的时候，联合索引很多时候可以满足覆盖索引，节省存储空间，避免回表，提高查询效率

要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率，因为涉及数据改变的时候，会维护索引的数据结构

如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含 NULL值时，它可以更好地确定哪个索引最有效地用于查询