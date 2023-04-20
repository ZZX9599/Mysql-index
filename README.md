# README

针对 mysql 的进阶知识，针对索引的方方面面，包括下面的内容：

- 存储引擎的介绍

- 索引的结构【着重是各类的数据结构和优缺点】

  - 索引的结构分类
  - 索引的支持情况

  - 二叉树
  - 二叉排序树
  - 平衡二叉树
  - 红黑树
  - B 树
  - B+ 树
  - Mysql 优化之后的B+ 树
  - Hash

- 索引的分类

  - 常见的分类

  - 聚集索引
  - 二级索引
  - 分析执行 SQL 语句

- 索引的语法

  - 创建索引
  - 查看索引
  - 删除索引

- 性能分析

  - SQL 的执行频率

  - 慢查询日志
  - SQL 的执行计划【重要】

- 索引的使用

  - 最左前缀法则
  - 联合索引范围查询
  - 索引失效的情况
  - 覆盖索引
  - 前缀索引
  - 单列索引和联合索引
  - 索引的设计原则