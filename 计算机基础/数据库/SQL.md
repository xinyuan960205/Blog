# SQL
## 一、SQL简介
### 1、什么是SQL
Structured Query Language：结构化查询语言

其实就是定义了操作所有关系型数据库的规则。每一种数据库操作的方式存在不一样的地方，称为“方言”。

### 2、SQL通用语法
1. SQL 语句可以单行或多行书写，以分号结尾。
2. 可使用空格和缩进来增强语句的可读性。
3. MySQL 数据库的 SQL 语句不区分大小写，关键字建议使用大写。
4. 3 种注释
	* 单行注释: -- 注释内容 或 # 注释内容(mysql 特有) 
	* 多行注释: /* 注释 */

### 3、SQL分类
1. DDL(Data Definition Language)数据定义语言
	用来定义数据库对象：数据库，表，列等。关键字：create, drop,alter 等
2. DML(Data Manipulation Language)数据操作语言
	用来对数据库中表的数据进行增删改。关键字：insert, delete, update 等
3. DQL(Data Query Language)数据查询语言
	用来查询数据库中表的记录(数据)。关键字：select, where 等
4. DCL(Data Control Language)数据控制语言(了解)
	用来定义数据库的访问权限和安全级别，及创建用户。关键字：GRANT， REVOKE 

## 二、DDL:操作数据库、表
CRUD
### 2.1 操作数据库
1. C(Create):创建
	* 创建数据库：
		* create database 数据库名称;
	* 创建数据库，判断不存在，再创建：
		* create database if not exists 数据库名称;
	* 创建数据库，并指定字符集
		* create database 数据库名称 character set 字符集名;
	* 练习： 创建db4数据库，判断是否存在，并制定字符集为gbk
		* create database if not exists db4 character set gbk;
2. R(Retrieve)：查询
	* 查询所有数据库的名称:
		* show databases;
	* 查询某个数据库的字符集:查询某个数据库的创建语句
		* show create database 数据库名称;
3. U(Update):修改
	* 修改数据库的字符集
		* alter database 数据库名称 character set 字符集名称;
4. D(Delete):删除
	* 删除数据库
		* drop database 数据库名称;
	* 判断数据库存在，存在再删除
		* drop database if exists 数据库名称;
5. 使用数据库
	* 查询当前正在使用的数据库名称
		* select database();
	* 使用数据库
		* use 数据库名称;



### 2.2 操作数据表
1. C(Create):创建
	1. 语法：
	```sql
		create table 表名(
			列名1 数据类型1,
			列名2 数据类型2,
			....
			列名n 数据类型n
		);
	```
		* 注意：最后一列，不需要加逗号（,）
		* 数据库类型：
			1. int：整数类型
				* age int,
			2. double:小数类型
				* score double(5,2)
			3. date:日期，只包含年月日，yyyy-MM-dd
			4. datetime:日期，包含年月日时分秒	 yyyy-MM-dd HH:mm:ss
			5. timestamp:时间错类型	包含年月日时分秒	 yyyy-MM-dd HH:mm:ss	
				* 如果将来不给这个字段赋值，或赋值为null，则默认使用当前的系统时间，来自动赋值
		
			6. varchar：字符串
				* name varchar(20):姓名最大20个字符
				* zhangsan 8个字符  张三 2个字符

	2. 创建表
	```sql
		create table student(
			id int,
			name varchar(32),
			age int ,
			score double(4,1),
			birthday date,
			insert_time timestamp
		);
	```
	3. 复制表：
		* `create table 表名 like 被复制的表名;`	  	
2. R(Retrieve)：查询
	* 查询某个数据库中所有的表名称
		* show tables;
	* 查询表结构
		* desc 表名;
3. U(Update):修改
	1. 修改表名
		alter table 表名 rename to 新的表名;
	2. 修改表的字符集
		alter table 表名 character set 字符集名称;
	3. 添加一列
		alter table 表名 add 列名 数据类型;
	4. 修改列名称 类型
		alter table 表名 change 列名 新列名 新数据类型;
		alter table 表名 modify 列名 新数据类型;
	5. 删除列
		alter table 表名 drop 列名;
4. D(Delete):删除
	* drop table 表名;
	* drop table  if exists 表名 ;


## 三、DML：增删改表中数据
### 3.1 添加数据：
* 语法：
	* `insert into 表名(列名1,列名2,...列名n) values(值1,值2,...值n);`
* 注意：
	1. 列名和值要一一对应。
	2. 如果表名后，不定义列名，则默认给所有列添加值<br>
		`insert into 表名 values(值1,值2,...值n);`
	3. 除了数字类型，其他类型需要使用引号(单双都可以)引起来

### 3.2 删除数据：
* 语法：
	* `delete from 表名 [where 条件]`
* 注意：
	1. 如果不加条件，则删除表中所有记录。
	2. 如果要删除所有记录
		1. `delete from 表名;` -- 不推荐使用。有多少条记录就会执行多少次删除操作
		2. `TRUNCATE TABLE 表名;` -- 推荐使用，效率更高 先删除表，然后再创建一张一样的表。

### 3.3 修改数据：
* 语法：
	* `update 表名 set 列名1 = 值1, 列名2 = 值2,... [where 条件];`
* 注意：
	1. 如果不加任何条件，则会将表中所有记录全部修改。

## 四、DQL：查询表中的记录
* `select * from 表名;`

### 1. 语法：
```
    select
    	字段列表
    from
    	表名列表
    where
    	条件列表
    group by
    	分组字段
    having
    	分组之后的条件
    order by
    	排序
    limit
    	分页限定
```
### 2. 基础查询
1. 多个字段的查询<br>
	`select 字段名1，字段名2... from 表名；`
	* 注意：
		* 如果查询所有字段，则可以使用*来替代字段列表。
2. 去除重复：
	* distinct
	* `select distinct 字段名1，字段名2... from 表名；`
3. 计算列
	* 一般可以使用四则运算计算一些列的值。（一般只会进行数值型的计算）
	* ifnull(表达式1,表达式2)：null参与的运算，计算结果都为null
		* 表达式1：哪个字段需要判断是否为null
		* 如果该字段为null后的替换值。
4. 起别名：
	* as：as也可以省略，加一个空格即可

### 3. 条件查询
1. where子句后跟条件
2. 运算符
	* > 、< 、<= 、>= 、= 、<>
	* BETWEEN...AND  
	* IN( 集合) 
	* LIKE：模糊查询
		* 占位符：
			* _:单个任意字符
			* %：多个任意字符
	* IS NULL  
	* and  或 &&
	* or  或 || 
	* not  或 !
3. 示例	
- 查询年龄大于20岁
```
SELECT * FROM student WHERE age > 20;
SELECT * FROM student WHERE age >= 20;
```

- 查询年龄等于20岁
```
SELECT * FROM student WHERE age = 20;
```

- 查询年龄不等于20岁
```
SELECT * FROM student WHERE age != 20;
SELECT * FROM student WHERE age <> 20;
```

- 查询年龄大于等于20 小于等于30
```
SELECT * FROM student WHERE age >= 20 &&  age <=30;
SELECT * FROM student WHERE age >= 20 AND  age <=30;
SELECT * FROM student WHERE age BETWEEN 20 AND 30;
```

- 查询年龄22岁，18岁，25岁的信息
```
SELECT * FROM student WHERE age = 22 OR age = 18 OR age = 25
SELECT * FROM student WHERE age IN (22,18,25);
```

- 查询英语成绩为null
```
SELECT * FROM student WHERE english = NULL; -- 不对的。null值不能使用 = （!=） 判断

SELECT * FROM student WHERE english IS NULL;
```

- 查询英语成绩不为null
```
SELECT * FROM student WHERE english  IS NOT NULL;
```

- 查询姓马的有哪些？ like
```
SELECT * FROM student WHERE NAME LIKE '马%';
```

- 查询姓名第二个字是化的人
```
SELECT * FROM student WHERE NAME LIKE "_化%";
```

- 查询姓名是3个字的人
```
SELECT * FROM student WHERE NAME LIKE '___';
```

- 查询姓名中包含德的人
```
SELECT * FROM student WHERE NAME LIKE '%德%';
```
### 4. 排序查询
* 语法：order by 子句
	* order by 排序字段1 排序方式1 ，  排序字段2 排序方式2...

* 排序方式：
	* ASC：升序，默认的。
	* DESC：降序。

* 注意：
	* 如果有多个排序条件，则当前边的条件值一样时，才会判断第二条件。
```sql
select * from student ORDER BY math ASC, english Desc;
```

### 5. 聚合函数
1. count：计算个数
	1. 一般选择非空的列：主键
	2. count(*)
```sql
SELECT COUNT(english) FROM student;
SELECT COUNT(IFNULL(english, 0)) FROM student;
SELECT COUNT(*) FROM student;
```
2. max：计算最大值
3. min：计算最小值
4. sum：计算和
5. avg：计算平均值

* 注意：聚合函数的计算，排除null值。
	解决方案：
		1. 选择不包含非空的列进行计算
		2. IFNULL函数

### 6. 分组查询
1. 语法：group by 分组字段；
2. 注意：
	1. 分组之后查询的字段：分组字段、聚合函数
	2. where 和 having 的区别？
		1. where 在分组之前进行限定，如果不满足条件，则不参与分组。having在分组之后进行限定，如果不满足结果，则不会被查询出来
		2. where 后不可以跟聚合函数，having可以进行聚合函数的判断。

- 按照性别分组。分别查询男、女同学的平均分
```
SELECT sex , AVG(math) FROM student GROUP BY sex;
```

- 按照性别分组。分别查询男、女同学的平均分,人数
```
SELECT sex , AVG(math),COUNT(id) FROM student GROUP BY sex;
```

-  按照性别分组。分别查询男、女同学的平均分,人数 要求：分数低于70分的人，不参与分组
```
SELECT sex , AVG(math),COUNT(id) FROM student WHERE math > 70 GROUP BY sex;
```

-  按照性别分组。分别查询男、女同学的平均分,人数 要求：分数低于70分的人，不参与分组,分组之后。人数要大于2个人
```
SELECT sex , AVG(math),COUNT(id) FROM student WHERE math > 70 GROUP BY sex HAVING COUNT(id) > 2;

SELECT sex , AVG(math),COUNT(id) 人数 FROM student WHERE math > 70 GROUP BY sex HAVING 人数 > 2;
```

### 7.分页查询
1. 语法：limit 开始的索引,每页查询的条数;
2. 公式：开始的索引 = （当前的页码 - 1） * 每页显示的条数
	-- 每页显示3条记录 
```
	SELECT * FROM student LIMIT 0,3; -- 第1页
	
	SELECT * FROM student LIMIT 3,3; -- 第2页
	
	SELECT * FROM student LIMIT 6,3; -- 第3页
```
3. limit 是一个MySQL"方言"

### 8. 子查询

子查询中只能返回一个字段的数据。

可以将子查询的结果作为 WHRER 语句的过滤条件：

```sql
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
               FROM mytable2);
```

下面的语句可以检索出客户的订单数量，子查询语句会对第一个查询检索出的每个客户执行一次：

```SQL
SELECT cust_name, (SELECT COUNT(*)
                   FROM Orders
                   WHERE Orders.cust_id = Customers.cust_id)
                   AS orders_num
FROM Customers
ORDER BY cust_name;
```

### 9. 连接

连接用于连接多个表，使用 JOIN 关键字，并且条件语句使用 ON 而不是 WHERE。

连接可以替换子查询，并且比子查询的效率一般会更快。

可以用 AS 给列名、计算字段和表名取别名，给表名取别名是为了简化 SQL 语句以及连接相同表。

#### 内连接

内连接又称等值连接，使用 INNER JOIN 关键字。

```
SELECT A.value, B.value
FROM tablea AS A INNER JOIN tableb AS B
ON A.key = B.key;
```

可以不明确使用 INNER JOIN，而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来。

```
SELECT A.value, B.value
FROM tablea AS A, tableb AS B
WHERE A.key = B.key;
```

#### 自连接

自连接可以看成内连接的一种，只是连接的表是自身而已。

一张员工表，包含员工姓名和员工所属部门，要找出与 Jim 处在同一部门的所有员工姓名。

子查询版本

```
SELECT name
FROM employee
WHERE department = (
      SELECT department
      FROM employee
      WHERE name = "Jim");
```

自连接版本

```
SELECT e1.name
FROM employee AS e1 INNER JOIN employee AS e2
ON e1.department = e2.department
      AND e2.name = "Jim";
```

#### 自然连接

自然连接是把同名列通过等值测试连接起来的，同名列可以有多个。

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列。

```
SELECT A.value, B.value
FROM tablea AS A NATURAL JOIN tableb AS B;
```

#### 外连接

外连接保留了没有关联的那些行。分为左外连接，右外连接以及全外连接，左外连接就是保留左表没有关联的行。

检索所有顾客的订单信息，包括还没有订单信息的顾客。

```
SELECT Customers.cust_id, Orders.order_num
FROM Customers LEFT OUTER JOIN Orders
ON Customers.cust_id = Orders.cust_id;
```

customers 表：

| cust_id | cust_name |
| ------- | --------- |
| 1       | a         |
| 2       | b         |
| 3       | c         |

orders 表：

| order_id | cust_id |
| -------- | ------- |
| 1        | 1       |
| 2        | 1       |
| 3        | 3       |
| 4        | 3       |

结果：

| cust_id | cust_name | order_id |
| ------- | --------- | -------- |
| 1       | a         | 1        |
| 1       | a         | 2        |
| 3       | c         | 3        |
| 3       | c         | 4        |
| 2       | b         | Null     |

### 10. 组合查询

使用 **UNION** 来组合两个查询，如果第一个查询返回 M 行，第二个查询返回 N 行，那么组合查询的结果一般为 M+N 行。

每个查询必须包含相同的列、表达式和聚集函数。

默认会去除相同行，如果需要保留相同行，使用 UNION ALL。

只能包含一个 ORDER BY 子句，并且必须位于语句的最后。

```sql
SELECT col
FROM mytable
WHERE col = 1
UNION
SELECT col
FROM mytable
WHERE col =2;
```







