# Sql Dev Best Practice

[TOC]

# 超过三张表禁止 `join`

摘抄自《阿里巴巴Java开发手册》

> 超过三张表禁止 join。需要 join 的字段，数据类型保持绝对一致；多表关联时，保证被关联的字段需要有索引。
> 说明：即使双表 join 也要注意表索引、SQL性能。

超过三张表进行 join ，当数据规模较大时，对数据库的压力很大。

```sql
# 😭
select * from tag
  join tag_post on tag_post.tag_id = tag.id
  join post on tag_post.post_id = post.id
where tag.tag = 'mysql';

# 👍
select * from tag where tag = 'mysql';
select * from tag_post where tag_id = 1234;
select * from post where id in (123, 456, 567, 9989, 8909);
```

应该将写简单 sql （查询单表），将逻辑放在应用层，这样业务逻辑会更清晰，而且在应用层（内存）实现特定的 join 也容易得多。很多高性能的应用都会对关联查询进行分解。简单地，可以对每个表进行一次单表查询，然后将结果在应用程序中进行关联。关于分解关联查询的优势更多细节可参考《高性能Mysql》中的 6.3.3 节。

# MySQL 避免重复插入数据

1. insert ignore into

```sql
INSERT IGNORE INTO USER (u_id, NAME, gender)
VALUES
	('UUID', 'luca', 'MALE');
```

2. on duplicate key update

```sql
INSERT INTO USER (u_id, NAME, gender)
VALUES
	('UUID', 'luca', 'MALE') ON DUPLICATE KEY UPDATE NAME = 'luca',
	gender = 'MALE';
```

3. replace into

```sql
REPLACE INTO USER (u_id, NAME, gender)
VALUES
	('UUID', 'luca', 'MALE');
```

4. insert if not exists

```sql
INSERT INTO USER (u_id, NAME, gender) SELECT
	'UUID',
	'luca',
	'MALE'
FROM
	USER
WHERE
	NOT EXISTS (
		SELECT
			u_id
		FROM
			USER
		WHERE
			u_id = 'UUID'
	);
```
