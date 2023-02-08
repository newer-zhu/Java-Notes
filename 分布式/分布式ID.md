### MySQL

IO写入连续性好，分库分表复杂，数据泄漏，索引速度快

```sql
CREATE TABLE `sequence_id_generator` (
  `id` int(10) NOT NULL,
  `current_max_id` bigint(20) NOT NULL COMMENT '当前最大id',
  `step` int(10) NOT NULL COMMENT '号段的长度',
  `version` int(20) NOT NULL COMMENT '版本号',
  `biz_type`    int(20) NOT NULL COMMENT '业务类型',
   PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

```

使用

```sql
UPDATE sequence_id_generator SET current_max_id = 0+100, version=version+1 WHERE version = 0  AND `biz_type` = 101
SELECT `current_max_id`, `step`,`version` FROM `sequence_id_generator` where `biz_type` = 101
```

### Redis自增 

性能高，数据会丢失和泄漏

### UUID

无需中央控制器，生成快。索引效率差，占用空间多。IO随机性大。

### 雪花算法

不依赖外部组件，缺点是时钟回拨会导致重复

![image-20220317211832836](E:\学习笔记\typora\img\image-20220317211832836.png)
