# 基础语法

## 查询

SELECT 语法：

> select ------ 查询
>
> id,name ------ 列名
>
> from ------ 数据来源于哪一张表
>
> user\_table ------ 表名
>
> ```sql
> -- 查询user_table表的数据：
> SELECT
> 	id,
> 	user_name 
> FROM
> 	user_table
> ```

![查询](D:/proJect/java-notes/docs/sql/img_sql/查询.png)

## 常用语法

> 分页
>
> ```sql
> -- limit a,b  a表示查询数据的起始位置，b表示返回的数量。例如:LIMIT 5,10表示从第6行开始显示，显示10条记录 。
> SELECT id,user_name FROM user_table
> limit 0,10
> ```
>
> 分组
>
> ```sql
> -- GROUP BY：对某一列进行分组
> -- GROUP_CONCAT：将分组中的字符连接成一个字段
> SELECT
> 	GROUP_CONCAT(user_name) userName,
> 	sex
> FROM user_table
> GROUP BY sex
> ```
>
> 计数
>
> ```sql
> -- count() 统计返回结果的行数
> SELECT COUNT(*) FROM user_table
> 
> -- sum() 对列的值进行求和
> SELECT SUM(id) FROM user_table
> ```
>
> 排序
>
> ```sql
> -- order by 对字段进行排序，desc表示倒序，asc表示正序，默认asc。
> SELECT id,user_name,sex FROM user_table
> ORDER BY id DESC
> ```
>
> 查询条件
>
> ```sql
> -- where 对条件进行筛选
> SELECT id,user_name,sex FROM user_table
> WHERE user_name = '张三'
> -- or 或者
> SELECT id,user_name,sex FROM user_table
> WHERE user_name = '张三' or user_name = '李四'
> -- like 模糊查询
> SELECT id,user_name,sex FROM user_table
> WHERE user_name like '张%'
> -- is not null 不为空
> SELECT id,user_name,sex FROM user_table
> WHERE user_name is not null
> -- is null 为空 
> SELECT id,user_name,sex FROM user_table
> WHERE user_name is null
> ```
>
> 左关联表
>
> ```sql
> -- left join 左关联。from后面查询主表，left join关联表 on后面表示关联条件
> -- 一个用户管理多个客户，查询能够管理到客户名称为‘周扒皮’的用户
> SELECT
> 	u.id,
> 	u.user_name,
> 	u.sex 
> FROM
> 	user_table u
> 	LEFT JOIN crm_table c ON c.user_id = u.id 
> WHERE
> 	c.crm_name = '周扒皮'
> ```



# 创建临时表

#### 临时表的作用

数据库临时表是一种特殊类型的表，在数据库中存储临时数据或者临时结果集；

临时表与永久表相似，但临时表存储在tempdb中，当不再使用时会自动删除。

临时表分本地临时表与全局临时表。

> 本地临时表，只能在当前查询页面使用，新开查询是不能使用它的。
>
> 全局临时表，不管开多少查询页面均可使用（一般用不到）。 



#### 本地临时表

```mysql
-- 创建一个临时表
CREATE TEMPORARY TABLE temp_table_name (
  id INT AUTO_INCREMENT,
  data VARCHAR(100),
  PRIMARY KEY (id),
  INDEX idx_data (data)
);
 
-- 向临时表中插入数据
INSERT INTO temp_table_name (data) VALUES ('org1'), ('org2');
 
-- 查询临时表中的数据
SELECT * FROM temp_table_name;
 
-- 例如，如果您想在一个复杂的查询中使用这个临时表，可以这样做：
SELECT t1.*, t2.*
FROM temp_table_name t1
JOIN another_table t2 ON t1.id = t2.temp_id;
 
-- 结束会话时，临时表会自动消失
```

