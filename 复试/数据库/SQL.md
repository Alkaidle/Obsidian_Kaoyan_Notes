#### 表操作
- 创建
```c
CREATE TABLE
[IF NOT EXISTS] tb_name -- 不存在才创建，存在就跳过
(column_name1 data_type1 -- 列名和类型必选
  [ PRIMARY KEY -- 可选的约束，主键
   | FOREIGN KEY -- 外键，引用其他表的键值
   | AUTO_INCREMENT -- 自增ID
   | COMMENT comment -- 列注释（评论）
   | DEFAULT default_value -- 默认值
   | UNIQUE -- 唯一性约束，不允许两条记录该列值相同
   | NOT NULL -- 该列非空
  ], ...
) [CHARACTER SET charset] -- 字符集编码
[COLLATE collate_value] -- 列排序和比较时的规则（是否区分大小写等）
```
从另一张表复制表结构创建表： `CREATE TABLE tb_name LIKE tb_name_old`
从另一张表的查询结果创建表： `CREATE TABLE tb_name AS SELECT * FROM tb_name_old WHERE options`
- 修改
```c
ALTER TABLE 表名 { ADD COLUMN <列名> <类型> [after colname]  -- 增加列
     | CHANGE COLUMN <旧列名> <新列名> <新列类型> -- 修改列名或类型
     | ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT } -- 修改/删除列的默认值
     | MODIFY COLUMN <列名> <类型> -- 修改列类型
     | DROP COLUMN <列名> -- 删除列
     | RENAME TO <新表名> -- 修改表名
     | CHARACTER SET <字符集名> -- 修改字符集
     | COLLATE <校对规则名> } -- 修改校对规则（比较和排序时用到）

```
- 索引
```c
-- 创建
CREATE 
  [UNIQUE -- 唯一索引
  | FULLTEXT -- 全文索引
  ] INDEX index_name ON table_name -- 不指定唯一或全文时默认普通索引
  (column1[(length) [DESC|ASC]] [,column2,...]) -- 可以对多列建立组合索引  

-- 删除
DROP INDEX <索引名> ON <表名>

-- 可用 ALTER 创建/删除
```
#### 数据操作
```mysql
-- 插入
INSERT INTO tablename (col1, col2) VALUES (val1, val2);
REPLACE INTO 若存在则先删除
-- 更新
UPDATE tablename SET col1=..., col2=... WHERE ...
-- 删除
DELETE FROM tablename WHERE col1=... 
-- 不改变结构，仅删除内容
TRUNCATE tb_name
```
#### 字符串
```sql
-- sub在str中第一次出现的位置, 没有则0
locate(sub, str) 
instr(str, sub)
like/ not like “%sub%”
```
- `WHERE` 过滤行，不可使用聚合函数，`GROUP BY` 前；`HAVING` 过滤组，可使用聚合函数，`GROUP BY` 后
- `from (select ...) t` 必须有别名
- “如果 `personId` 的地址不在 `Address` 表中，则报告为 `null`” → 主表为`personId`，左连接
- <u>如何查找第N高的数据</u>
	- `limit n` 返回前n条数据
	- `offset n` 跳过n条语句
	- 等价于 `limit offset, size`
	- 判断空值：`ifnull(a,b)` 如果a非空返回a，否则返回b
	- `limit` 返回的是整个结果而非分组的前n行
- <u>排名</u>
	- `RANK() OVER (PARTITION BY 列 ORDER BY 列)`
	- `RANK()` 会跳号：1,2,2,4	
	- `DENSE_RANK()` 不会跳号：1,2,2,3
	- `ROW_NUMBER()` 不允许并列：1,2,3,4
	- **注意**：窗口函数 只能在 select 或者 order by 后面
- <u>增加日期/时间</u>
	- `DATE_ADD(date, INTERVAL 1 DAY/MONTH/HOUR)`
- <u>唯一</u>
	- `count(*) over (partition over...)=1`
	- 不存在 id不同且相等
- <u>不唯一</u>
	- `>1`
	- 存在 id不同且相等
#### 日期
`DATEDIFF(date1, date2)` 返回天数，忽略时间，只比较日期
`TIMESTAMPDIFF(interval, time1, time2)`  interval 可以是不同的时间单位
#### 查询
##### 截断平均值
```mysql
round((sum(score)-max(score)-min(score)) / (count(score)-2), 1)
```
注意不能`count(*)`，因为会将`NULL`算入，`count(colname)`自动去除重复
##### 去重组合统计
**统计唯一组合**
```mysql
distinct (uid, date(submit_time)) -- MySQL支持

-- 等价于
select count(*)
from (
    select uid, date(submit_time)
    from exam_record
    group by uid, date(submit_time)
) t
``` 
![[Pasted image 20260312195819.png]]
