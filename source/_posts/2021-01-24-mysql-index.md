---
title: Mysql 索引
date: 2021-01-24 13:44:22
tags:
  - mysql
---

## Mysql索引
索引的出现其实就是为了提⾼数据查询的效率

MySQL 在查询⽅⾯主要就是两种⽅式：
+ 全表扫描（⼀个⼀个挨个找）
+ 根据索引检索

### 建⽴索引注意事项
+ 索引不是越多越好，虽然索引会提⾼ select 效率，但是也降低了insert以及update的效率
+ 数据量⼩的表不需要建⽴索引，会增加额外的索引开销
+ 不经常使⽤的列不要建⽴索引
+ 频繁更新的列不要建⽴索引，会影响更新的效率

### MySQL的索引有⼏种
+ 普通索引：最基本的索引，没有任何限制
+ 唯⼀索引：与普通索引类似，但索引列的值必须是唯⼀的，允许空值
+ 主键索引：⼀种特殊的唯⼀索引，⼀个表只能有⼀个主键，不允许有空值
+ 组合索引：在多个字段上创建的索引，只有在查询条件中使⽤了创建索引的第⼀个字段，索引才会被使⽤
+ 全⽂索引：主要⽤来查找⽂本中的关键字，类似于搜索引擎

### 索引优化
+ 尽量避免在where字句中对字段进⾏空值判断，这会导致引擎放弃使⽤索引，进⾏全表扫描
+ 字段值分布很稀少的字段，不适合建⽴索引
+ 不要⽤字符字段做主键
+ 字符字段只建⽴前缀索引
+ 不要⽤外键和UNIQUE
+ 使⽤多列索引时，注意顺序和查询条件保持⼀致，同时删除不必要的单列索引

### 索引失效的情况
+ 模糊匹配当中以 % 开头时，索引失效
+ OR 有⼀边的条件字段没有索引时，索引失效
+ 使⽤复合索引的时候，没有使⽤左侧的列查找，索引失效
+ 在 where 当中索引列参加了运算，索引失效
+ 在 where 当中索引列使⽤了函数，索引失效

> Note:
> 1. 在任何数据库当中主键上都会⾃动添加索引对象
> 2. 在mysql当中，⼀个字段上如果有unique约束的话，也会⾃动创建索引对象