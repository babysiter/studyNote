@[TOC](这里写目录标题)
# 表
### 创建表

CREATE [TEMPORARY] TABLE[ IF NOT EXISTS] [库名.]表名 ( **表的内部结构定义** )[ **表选项**]
* TEMPORARY 临时表，会话结束时表自动消失


#### 表的内部结构定义
```
对于表内字段的定义（每个[]内代表一个字段选项，可写可不写）：
字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']

- PRIMARY 主键
  - 能唯一标识记录的字段，可以作为主键。
  - 一个表只能有一个主键。
  -  声明字段时，用 primary key 标识。
        也可以在字段列表之后声明
        例：create table tab ( id int, stu varchar(10), primary key (id));
  - 主键字段的值不能为null。
  - 主键可以由多个字段共同组成。此时需要在字段列表后声明的方法。
     例：create table tab ( id int, stu varchar(10), age int, primary key (stu, age));
- NIQUE 唯一索引
	使得某字段的值也不能重复。
- NULL 约束
 null不是数据类型，是列的一个属性。
 null, 允许为空。默认。
 not null, 不允许为空。
 insert into tab values (null, 'val');
   -- 此时表示将第一个字段的值设为null, 能否运行成功取决于该字段是否允许为null
- DEFAULT 默认值属性
 insert into tab values (default, 'val');  —— 此时表示强制使用默认值。
 create table tab ( add_time timestamp default current_timestamp ); —— 表示将当前时间的时间戳设为默认值。
- AUTO_INCREMENT 自动增长约束
	* 自动增长必须为索引（主键或unique）
    * 一个表只能存在一个字段为自动增长。
    * 默认为1开始自动增长。可以通过表属性 auto_increment = x进行设置，或 alter table tbl auto_increment = x;
- COMMENT 注释
	例：create table tab ( id int ) comment '注释内容';
- FOREIGN KEY 外键约束
  用于限制主表与从表数据完整性。
    alter table t1 add constraint `t1_t2_fk` foreign key (t1_id) references t2(id);
	 - 将表t1的t1_id外键关联到表t2的id字段。
     - 每个外键都有一个名字，可以通过 constraint 指定
   存在外键的表，称之为从表（子表），外键指向的表，称之为主表（父表）。
   作用：保持数据一致性，完整性，主要目的是控制存储在外键表（从表）中的数据。
	MySQL中，可以对InnoDB引擎使用外键约束：
    foreign key (外键字段） references 主表名 (关联字段) [主表记录删除时的动作] [主表记录更新时的动作]
    此时需要检测一个从表的外键需要约束为主表的已存在的值。外键在没有关联的情况下，可以设置为null.前提是该外键列，没有not null。
    可以不指定主表记录更改或更新时的动作，那么此时主表的操作被拒绝。
    如果指定了 on update 或 on delete：在删除或更新时，有如下几个操作可以选择：
    1. cascade，级联操作。主表数据被更新（主键值更新），从表也被更新（外键值更新）。主表记录被删除，从表相关记录也被删除。
    2. set null，设置为null。主表数据被更新（主键值更新），从表的外键被设置为null。主表记录被删除，从表相关记录外键被设置成null。但注意，要求该外键列，没有not null属性约束。
    3. restrict，拒绝父表删除和更新。
	注意，外键只被InnoDB存储引擎所支持。其他引擎是不支持的。
```
#### 表选项
```
-- 字符集
  CHARSET = charset_name
  如果表没有设定，则使用数据库字符集
-- 存储引擎
  ENGINE = engine_name
>   表在管理数据时采用的不同的数据结构，结构不同会导致处理方式、提供的特性操作等不同
  常见的引擎：InnoDB、 MyISAM、 Memory/Heap 、BDB、 Merge、 Example、 CSV、 MaxDB Archive,不同的引擎在保存表的结构和数据时采用不同的方式.
  SHOW ENGINES -- 显示存储引擎的状态信息
  SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
  -- 数据文件目录
        DATA DIRECTORY = '目录'
  -- 索引文件目录
        INDEX DIRECTORY = '目录'
  -- 表注释
        COMMENT = 'string'
  -- 分区选项
        PARTITION BY ... (详细见手册)
  ```
  详细了解可以参考这个博客
  [Mysql常见的数据库引擎以及对比](https://www.cnblogs.com/wangjian941118/p/10421716.html)
### 查看所有表
```
	SHOW TABLES[ LIKE 'pattern'] 
	SHOW TABLES FROM 数据库名
    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern'] 直接打印各个表格的状态包括数据，信息更加详细
```
### 查看表的内部结构定义
```
	SHOW CREATE TABLE 表名
    DESC 表名
    DESCRIBE 表名
    EXPLAIN  表名
    SHOW COLUMNS [FROM 表名]|[like 'Pattern']
    SHOW TABLE STATUS

```
## 修改表的内部结构和表选项

--修改表结构和表选项
  ALTER TABLE 表名 **[表的选项] [表的操作]**
 **其中表的选项可以参考文章前面部分中的内容**
 RENAME TABLE 原表名 TO 新表名
-- 对表进行重命名
    RENAME TABLE 原表名 TO 库名.表名 （可将表移动到另一个数据库）

 #### 表的操作
 ```
-- 增加字段
 ADD[ COLUMN] 字段定义 [AFTER 字段名]|[FIRST]
     - AFTER 字段名       -- 表示增加在该字段名后面
     - FIRST           -- 表示增加在第一个
-- 创建主键
 ADD PRIMARY KEY(字段名)   -- 创建主键
-- 创建唯一索引
 ADD UNIQUE [索引名] (字段名]
-- 创建普通索引
 ADD INDEX [索引名] (字段名)
-- 删除字段
 DROP[ COLUMN] 字段名
-- 支持对字段属性进行修改，不能修改字段名(所有原有属性也需写上)
  MODIFY[ COLUMN] 字段名 字段属性
-- 支持对字段名修改
  CHANGE[ COLUMN] 原字段名 新字段名 字段属性
-- 删除主键(删除主键前需删除其AUTO_INCREMENT属	性)
  DROP PRIMARY KEY 
-- 删除索引
  DROP INDEX 索引名 
-- 删除外键
  DROP FOREIGN KEY 外键
```
### 其他表操作
```
-- 删除表
    DROP TABLE[ IF EXISTS] 表名 ...

-- 清空表数据
    TRUNCATE [TABLE] 表名

-- 复制表结构
    CREATE TABLE 表名 LIKE 要复制的表名

-- 复制表结构和数据
    CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名

-- 检查表是否有错误
    CHECK TABLE tbl_name [, tbl_name] ... [option] ...

-- 优化表
   OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

-- 修复表
   REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]

-- 分析表
   ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name]
 ```
# 数据库
### 创建库
```
CREATE DATABASE[ IF NOT EXISTS] 数据库名 数据库选项
--数据库选项：
    CHARACTER SET charset_name ——字符编码
    COLLATE collation_name ——简而言之，COLLATE会影响到ORDER BY语句的顺序，会影响到WHERE条件中大于小于号筛选出来的结果，会影响DISTINCT、GROUP BY、HAVING语句的查询结果。另外，mysql建索引的时候，如果索引列是字符类型，也会影响索引创建，只不过这种影响我们感知不到。总之，凡是涉及到字符类型比较或排序的地方，都会和COLLATE有关。
    详细了解可以查看博客[collate是什么](https://www.cnblogs.com/qcloud1001/p/10033364.html)

```
### 其他库操作
```

-- 查看当前数据库
SELECT DATABASE();
 
-- 显示当前时间、用户名、数据库版本
SELECT now(), user(), version();
-- 查看已有库
    SHOW DATABASES[ LIKE 'PATTERN']
 
-- 查看当前库信息
    SHOW CREATE DATABASE 数据库名
 
-- 修改库的选项信息
    ALTER DATABASE 库名 选项信息
 
-- 删除库
    DROP DATABASE[ IF EXISTS] 数据库名
        同时删除该数据库相关的目录及其目录内容

```
## 字符集编码
**MySQL本机、数据库、表、字段**均可设置编码
数据编码与客户端编码不需一致
```
SHOW VARIABLES LIKE 'character_set_%'   -- 查看所有本机Mysql字符集编码项
    character_set_client        客户端向服务器发送数据时使用的编码
    character_set_results       服务器端将结果返回给客户端所使用的编码
character_set_connection    连接层编码
SET 变量名 = 变量值
    SET character_set_client = gbk;
    SET character_set_results = gbk;
    SET character_set_connection = gbk;
SET NAMES GBK;  -- 相当于完成以上三个设置

SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']   查看所有字符集
SHOW COLLATION [LIKE 'pattern']     查看所有校对集（校对集用以排序详情查看[collate是什么](https://www.cnblogs.com/qcloud1001/p/10033364.html)）
```
## 列的数据类型
### 数值类型
- 默认存在符号位，在类型前加unsigned 无符号位。
- 显示宽度，如果某个数不够定义字段时设置的位数，则前面以0补填，zerofill 属性修改，例：int(5)   插入一个数'123'，补填后为'00123'，在满足要求的情况下，越小越好
- 1表示bool值真，0表示bool值假。MySQL没有布尔类型，通过整型0和1表示。常用tinyint(1)表示布尔型。

#### 整形
```

	tinyint     1字节    -128 ~ 127      无符号位：0 ~ 255
    smallint    2字节    -32768 ~ 32767
    mediumint   3字节    -8388608 ~ 8388607
    int         4字节
    bigint      8字节
    int(M)  M表示总位数
   
```
#### 浮点型
```
 float(单精度)     4字节
 double(双精度)    8字节
 不同于整型，浮点型前后均会补填0,定义浮点型时，需指定总位数和小数位数。
  float(M, D)     double(M, D)
    - M表示总位数，D表示小数位数。
    - M和D的大小会决定浮点数的范围。不同于整型的固定范围。
    - M既表示总位数（不包括小数点和正负号），也表示显示宽度（所有显示符号均包括）。
```
#### 定点数
```
 decimal -- 可变长度
    decimal(M, D)   M也表示总位数，D表示小数位数。
    保存一个精确的数值，不会发生数据的改变，不同于浮点数的四舍五入。
    将浮点数转换为字符串来保存，每9位数字保存为4个字节。
```
### 字符串类型
####  char, varchar
```
	char(M)    定长字符串，速度快，但浪费空间
    varchar(M) 变长字符串，速度慢，但节省空间
    M表示能存储的最大长度，此长度是字符数，非字节数。不同的编码，所占用的空间不同。
	char,最多255个字符，与编码无关。
    varchar,最多65535字符，与编码有关。
    一条有效记录最大不能超过65535个字节，但最大有效数据时65532字节因为在varchar存字符串时，第一个字节是空的，不存在任何数据，然后还需两个字节来存放字符串的长度，所以有效长度是64432-1-2=65532字节。。
    utf8 最大为21844个字符，gbk 最大为32766个字符，latin1 最大为65532个字符
    varchar 是变长的，需要利用存储空间保存 varchar 的长度，如果数据小于255个字节，则采用一个字节来保存长度，反之需要两个字节来保存。

```
####  blob, text、binary
```
 blob 二进制字符串（字节字符串）
        tinyblob, blob, mediumblob, longblob
 text 非二进制字符串（字符字符串）
        tinytext, text, mediumtext, longtext
    text 在定义时，不需要定义长度，也不会计算总长度。
    text 类型在定义时，不可给default值
 binary, varbinary
 	  类似于char和varchar，用于保存二进制字符串，也就是保存字节字符串而非字符字符串。
     char, varchar, text 对应 binary, varbinary, blob.

```
#### 日期时间类型
```
- datetime    8字节    日期及时间     1000-01-01 00:00:00 到 9999-12-31 23:59:59
- date        3字节    日期         1000-01-01 到 9999-12-31
- timestamp   4字节    时间戳        1970-01-01 00:00:00 到 2038-01-19 03:14:07
- time        3字节    时间         -838:59:59 到 838:59:59
- year        1字节    年份         1901 - 2155

```
#### 枚举和集合
```
-- 枚举(enum)
	enum(val1, val2, val3...)，在已知的值中进行单选。最大数量为65535。
    枚举值在保存时，以2个字节的整型(smallint)保存。每个枚举值，按保存的位置顺序，从1开始逐一递增。
    表现为字符串类型，存储却是整型。
    NULL值的索引是NULL
    空字符串错误值的索引值是0。
-- 集合（set）
	create table tab ( gender set('男', '女', '无') );
    insert into tab values ('男, 女');
    最多可以有64个不同的成员。以bigint存储，共8个字节。采取位运算的形式。
当创建表时，SET成员值的尾部空格将自动被删除。
```
set数据集可以参考以下博客[set的使用](https://blog.csdn.net/wrh_csdn/article/details/79739852)
##数据操作
### SELECT查询
```
-- 语法
	SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY [合计函数] -> HAVING -> ORDER BY -> LIMIT
-- a. select_expr
	-- 可以用 * 表示所有字段。
        select * from tb;
    -- 可以使用表达式（计算公式、函数调用、字段也是个表达式）
        select stu, 29+25, now() from tb;
    -- 可以为每个列使用别名。适用于简化列标识，避免多个列标识符重复。
        - 使用 as 关键字，也可省略 as.
        select tb_name [as] tn from tb;

```
### from语句
```
用于标识查询来源。
    -- 可以为表起别名。使用as关键字。
        SELECT * FROM tb1 AS tt, tb2 AS bb;
    -- from子句后，可以同时出现多个表。
        -- 多个表会横向叠加到一起，而数据会形成一个笛卡尔积。
        SELECT * FROM tb1, tb2;
 ```
 

### WHERE 子句

 -- 从from获得的数据源中进行筛选。
    -- 整型1表示真，0表示假。
    -- 表达式由运算符和运算数组成。
      -- 运算数：变量（字段）、值、函数返回值
      -- 运算符：
          =, <=>, <>, !=, <=, <, >=, >, !, &&, ||,
          in (not) null, (not) like, (not) in, (not) between and, is (not), and, or, not, xor
          is/is not 加上ture/false/unknown，检验某个值的真假
          <=>与<>功能相同，<=>可用于null比较

### GROUP BY 子句, 分组子句

-- 语法
	**GROUP BY 字段/别名 [排序方式]**
    分组后会进行排序。升序：ASC，降序：DESC
    以下[合计函数]需配合 GROUP BY 使用：
    	- count 返回不同的非NULL值数目  count(*)、count(字段)
    	- sum 求和
    	- max 求最大值
    	- min 求最小值
    	- avg 求平均值
    	- group_concat 返回带有来自一个组的连接的非NULL值的字符串结果。组内字符串连接。
-- SQL实例：显示每个地区的总人口数和总面积．
	SELECT region, SUM(population), SUM(area) FROM bbc GROUP BY region

### HAVING 子句，条件子句
#### where与having的区别
  * 与 where 功能、用法相同，执行时机不同。
 * where 在开始时执行检测数据，对原数据进行过滤。
  *  having 对筛选出的结果再次进行过滤。
  - having 字段必须是查询出来的，where 字段必须是数据表存在的。
   * where 不可以使用字段的别名，having 可以。因为执行WHERE代码时，可能尚未确定列值。
    * where 不可以使用合计函数。一般需用合计函数才会用 having
    * SQL标准要求HAVING必须引用GROUP BY子句中的列或用于合计函数中的列。
#### SQL实例：
显示每个地区的总人口数和总面积．仅显示那些面积超过1000000的地区。
  ```
    SELECT region, SUM(population), SUM(area)
    FROM bbc
    GROUP BY region
    HAVING SUM(area)>1000000
```
### ORDER BY 子句，排序子句

 order by 排序字段/别名 排序方式 [,排序字段/别名 排序方式]...
    升序：ASC，降序：DESC
    支持多个字段的排序。

###  LIMIT 子句，限制结果数量子句

 仅对处理好的结果进行数量限制。将处理好的结果的看作是一个集合，按照记录出现的顺序，索引从0开始。
    limit 起始位置, 获取条数
 省略第一个参数，表示从索引0开始。limit 获取条数

### DISTINCT/ALL 选项

distinct 去除重复记录
默认为 all, 全部记录

### UNION

将多个select查询的结果组合成一个结果集合。
**(SELECT ...) UNION [ALL|DISTINCT] (SELECT ...)**
默认 DISTINCT 方式，即所有返回的行都是唯一的
每个select查询的字段列表(数量、类型)应一致，因为结果中的字段
**建议:**
- 对每个SELECT查询加上小括号包裹。
- ORDER BY 排序时，需加上 LIMIT 进行结合。
## 联接（join）
 ### 显示联接的内联接（inner join）
   * 默认就是内联接
   *  SELECT * FROM tb1, tb2;
    上面的联接是一个隐式联接，因为SELECT语句中并没有join关键字。join是联接的关键字，可以通过join显式的指定一个联接。
    	SELECT name,laterNum FROM student INNER JOIN later ON (student.studentNO =later.studentNO)
    * 显式内联接与上面的隐式联接返回的最终结果集是一样的，只不过将联接条件放到了FROM子句中，而隐式联接中联接条件是放在WHERE子句中的。
  ### 显示联接的左外联接，右外联接
   左联接经常用在以下情况中，就是联接条件中，被联接的表中的联接列式联接表中联接列的子集。
    右外联接与左外联接正好相反，被联接的表的所有行必须出现在FROM子句的结果集中。所以将两个表对调一下，然后用右外联接与左外连接的结果是一样的。
    比如想查某个班的学生选的数据库这门课，有的同学没有选，但是也要把这个同学的那一列信息也显示出来就可以利用左外联接把学生表放在from后，把学生选课表放在left outer join 后。例如：
  ```
     select student.name,student_course.course_name
     from student left outer join student_course on (student.studentid=student_course.studentid and student_course.course_name='数据库')
   ```
   在显式联接中，如果联接条件的两个列名字相同，并且联接条件是二者相等，则可以用USING。
   ```	
    SELECT name,course_name FROM student LEFT OUTER JOIN student_course USING(studentid)
   ```
   ### 自然连接(natural join)
   相当于省略了using，会自动查找相同字段名。
      -  natural join
       - natural left join
       - natural right join
   ### 交叉连接 cross join
   即，没有条件的内连接。
        select * from tb1 cross join tb2;
   
### 子查询

子查询需用括号包裹。
**-- from型**
    from后要求是一个表，必须给子查询结果取个别名。
    - 简化每个查询内的条件。
    - from型需将结果生成一个临时表格，可用以原表的锁定的释放。
    - 子查询返回一个表，表型子查询。
    select * from (select * from tb where id>0) as subfrom where id>1;
**-- where型**
    - 子查询返回一个值，标量子查询。
    - 不需要给子查询取别名。
    - where子查询内的表，不能直接用以更新。
    select * from tb where money = (select max(money) from tb);
**-- 特殊运算符**
    != all()    相当于 not in
    = some()    相当于 in。any 是 some 的别名
    != some()   不等同于 not in，不等于其中某一个。
	all, some 可以配合其他运算符一起使用。

### INSERT

可以省略对列的指定，要求 values () 括号内，提供给了按照列顺序出现的所有字段的值。
    或者使用set语法。
    INSERT INTO tbl_name SET field=value,...；
    
可以一次性使用多个值，采用(), (), ();的形式。
    INSERT INTO tbl_name VALUES (), (), ();
    
可以在列值指定时，使用表达式。
    INSERT INTO tbl_name VALUES (field_value, 10+10, now());
    
可以使用一个特殊值 DEFAULT，表示该列使用默认值。
    INSERT INTO tbl_name VALUES (field_value, DEFAULT);
    
可以通过一个查询的结果，作为需要插入的值。
    INSERT INTO tbl_name SELECT ...;
    
可以指定在插入的值出现主键（或唯一索引）冲突时，更新其他非主键列的信息。
INSERT INTO tbl_name VALUES/SET/SELECT ON DUPLICATE KEY UPDATE 字段=值, …;

### DELETE

**DELETE FROM tbl_name [WHERE where_definition] [ORDER BY ...] [LIMIT row_count]**
按照条件删除。where
指定删除的最多记录数。limit
可以通过排序条件删除。order by + limit
支持多表删除，使用类似连接语法。

### TRUNCATE

**TRUNCATE [TABLE] tbl_name**
清空表内数据
TRUNCATE和delete的区别
	1，truncate 是删除表再创建，delete 是逐条删除
    2，truncate 重置auto_increment的值。而delete不会
    3，truncate 不知道删除了几条，而delete知道。
    4，当被用于带分区的表时，truncate 会保留分区


## 视图
###什么是视图：
   视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。
    视图具有表结构文件，但不存在数据文件。
    对其中所引用的基础表来说，视图的作用类似于筛选。定义视图的筛选可以来自当前或其它数据库的一个或多个表，或者其它视图。通过视图进行查询没有任何限制，通过它们进行数据修改时的限制也很少。
    视图是存储在数据库中的查询的sql语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，如：社会保险基金表，可以用视图只显示姓名，地址，而不显示社会保险号和工资数等，另一原因是可使复杂的查询易于理解和使用。
### 创建视图

**CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement**
    - 视图名必须唯一，同时不能与表重名。
    - 视图可以使用select语句查询到的列名，也可以自己指定相应的列名。
    - 可以指定视图执行的算法，通过ALGORITHM指定。
    - column_list如果存在，则数目必须等于SELECT语句检索的列数

### 查看结构
SHOW CREATE VIEW view_name
### 删除视图
- 删除视图后，数据依然存在。
 - 可同时删除多个视图。
    DROP VIEW [IF EXISTS] view_name ...
### 修改视图结构
一般不修改视图，因为不是所有的更新视图都会映射到表上。
    ALTER VIEW view_name [(column_list)] AS select_statement
### 视图作用
  1. 简化业务逻辑
  2. 对客户端隐藏真实的表结构

### 视图算法
 - MERGE       合并
    将视图的查询语句，与外部查询需要先合并再执行！
 - TEMPTABLE   临时表
    将视图执行完毕后，形成临时表，再做外层查询！
 - UNDEFINED   未定义(默认)，指的是MySQL自主去选择相应的算法。

## 事务(transaction)
   - **支持连续SQL的集体成功或集体撤销。**
   - **需要利用 InnoDB 或 BDB 存储引擎，对自动提交的特性支持完成。**
   - **InnoDB被称为事务安全型引擎。**
   - **事务开启**
    START TRANSACTION; 或者 BEGIN;
    开启事务后，所有被执行的SQL语句均被认作当前事务内的SQL语句。


- **事务的特性**
    1. 原子性（Atomicity）
        事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
    2. 一致性（Consistency）
        事务前后数据的完整性必须保持一致。
        - 事务开始和结束时，外部数据一致
        - 在整个事务过程中，操作是连续的
    3. 隔离性（Isolation）
        多个用户并发访问数据库时，一个用户的事务不能被其它用户的事物所干扰，多个并发事务之间的数据要相互隔离。
    4. 持久性（Durability）
        一个事务一旦被提交，它对数据库中的数据改变就是永久性的。
- **事务的原理**
    利用InnoDB的自动提交(autocommit)特性完成。
    普通的MySQL执行语句后，当前的数据提交操作均可被其他客户端可见。
    而事务是暂时关闭“自动提交”机制，需要commit提交持久化数据操作。
- **注意**
    1. 数据定义语言（DDL）语句不能被回滚，比如创建或取消数据库的语句，和创建、取消或更改表或存储的子程序的语句。
    2. 事务不能被嵌套
- **保存点**
    SAVEPOINT 保存点名称 -- 设置一个事务保存点
    ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
    RELEASE SAVEPOINT 保存点名称 -- 删除保存点
- **InnoDB自动提交特性设置**
    SET autocommit = 0|1;   0表示关闭自动提交，1表示开启自动提交。
    - 如果关闭了，那普通操作的结果对其他客户端也不可见，需要commit提交后才能持久化数据操作。
    - 也可以关闭自动提交来开启事务。但与START TRANSACTION不同的是，
        SET autocommit是永久改变服务器的设置，直到下次再次修改该设置。(针对当前连接)
        而START TRANSACTION记录开启前的状态，而一旦事务提交或回滚后就需要再次开启事务。(针对当前事务)

## 触发器
  触发程序是与表有关的命名数据库对象，当该表出现特定事件时，将激活该对象。包括记录的增加、修改、删除。
### 创建触发器

**CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt**
    参数：
   - trigger_time是触发程序的动作时间。它可以是 before 或 after，以指明触发程序是在激活它的语句之前或之后触发。
    - trigger_event指明了激活触发程序的语句的类型
        INSERT：将新行插入表时激活触发程序
        UPDATE：更改某一行时激活触发程序
        DELETE：从表中删除某一行时激活触发程序
    - tbl_name：监听的表，必须是永久性的表，不能将触发程序与TEMPORARY表或视图关联起来。
    - trigger_stmt：当触发程序激活时执行的语句。执行多个语句，可使用BEGIN...END复合语句结构,触发器语法规则与存储过程基本一致，语法规则请参考下文存储过程
   
	

###  删除触发器
**DROP TRIGGER [schema_name.]trigger_name**

## 存储过程
### 定义
**存储过程（Stored Procedure）**是在大型数据库系统中，一组为了完成特定功能的SQL 语句集，存储在数据库中，经过第一次编译后调用不需要再次编译，用户通过指定存储过程的名字并给出参数（如果该存储过程带有参数）来执行它。存储过程是数据库中的一个重要对象。
### 存储过程的特点
- 能完成较复杂的判断和运算
- 可编程行强，灵活
-  SQL编程的代码可重复使用
- 执行的速度相对快一些
- 减少网络之间的数据传输，节省开销 



### 编写存储过程
 (1)、变量的声明使用declare,一句declare只声明一个变量，变量必须先声明后使用；
 (2)、变量具有数据类型和长度，与mysql的SQL数据类型保持一致，因此甚至还能制定默认值、字符集和排序规则等；
 (3)、变量可以通过set来赋值，也可以通过select into的方式赋值；
 	例如：select * from tbl into var1,var2;
    select查询到的值必须和后面变量数量类型相同。
 (4)、变量需要返回，可以使用select语句，如：select 变量名。
 
### 变量的作用域
 **1、变量作用域说明：**
        (1)、存储过程中变量是有作用域的，作用范围在begin和end块之间，end结束变量的作用范围即结束。
        (2)、需要多个块之间传值，可以使用全局变量，即放在所有代码块之前
        (3)、传参变量是全局的，可以在多个块之间起作用
    **2、通过一个实例来验证变量的作用域**
       需求: 创建一个存储过程，用来统计表users、orders表中行数数量和orders表中的最大金额和最小金额
```
create procedure test3()
begin
  begin
    declare userscount int default 0; -- 用户表中的数量
    declare ordercount int default 0; -- 订单表中的数量
    select count(*) into userscount from users;
    select count(*) into ordercount from orders;
    select userscount,ordercount; -- 返回用户表中的数量、订单表中的数量
  end;
  begin 
    declare maxmoney int default 0; -- 最大金额
    declare minmoney int default 0; -- 最小金额
    select max(money) into maxmoney from orders;
    select min(money) into minmoney from orders;
    select maxmoney,minmoney; -- 返回最金额、最小金额
   end;
end;
```
调用以上存储过程不会出现问题。如果现在修改一下代码。
```
create procedure test3()
    begin
      begin
        declare userscount int default 0; -- 用户表中的数量
        declare ordercount int default 0; -- 订单表中的数量
        select count(*) into userscount from users;
        select count(*) into ordercount from orders;
        select userscount,ordercount; -- 返回用户表中的数量、订单表中的数量
      end;
      begin 
        declare maxmoney int default 0; -- 最大金额
        declare minmoney int default 0; -- 最小金额
        select max(money) into maxmoney from orders;
        select min(money) into minmoney from orders;
        select userscount,ordercount，maxmoney,minmoney; -- 增加了此处代码，返回最金额、最小金额
       end;
    end;
```
代码报错，无法识别userscount,ordercount
于是将userscount,ordercount修改为全局变量。
```
 create procedure test3()
    begin

        declare userscount int default 0; -- 用户表中的数量
        declare ordercount int default 0; -- 订单表中的数量
        begin
            select count(*) into userscount from users;
            select count(*) into ordercount from orders;
            select userscount,ordercount; -- 返回用户表中的数量、订单表中的数量
      end;
      begin 
        declare maxmoney int default 0; -- 最大金额
        declare minmoney int default 0; -- 最小金额
        select max(money) into maxmoney from orders;
        select min(money) into minmoney from orders;
        select userscount,ordercount，maxmoney,minmoney; -- 返回最金额、最小金额
       end;
    end;
```
运行成功
### 存储过程参数
```
create procedure 名称([IN|OUT|INOUT] 参数名 参数数据类型 )
begin
.........
end
```
#### 存储过程的传出参数IN
 说明：

        （1）、传入参数：类型为in,表示该参数的值必须在调用存储过程事指定，如果不显示指定为in,那么默认就是in类型。
        （2）、IN类型参数一般只用于传入，在调用过程中一般不作为修改和返回
        （3）、如果调用存储过程中需要修改和返回值，可以使用OUT类型参数
#### 存储过程的传出参数out
需求：调用存储过程时，传入userId返回该用户的name
```
 create procedure test5(in userId int,out username varchar(32))
  begin
  	select name into username from users where id=userId;
  end;
```
概括：
        1、传出参数：在调用存储过程中，可以改变其值，并可返回；
        2、out是传出参数，不能用于传入参数值；
        3、调用存储过程时，out参数也需要指定，但必须是变量，不能是常量；
        4、如果既需要传入，同时又需要传出，则可以使用INOUT类型参数

#### 存储过程的可变参数INOUT

需求：调用存储过程时，传入userId和userName,即使传入，也是传出参数。
```
create procedure test6(inout userId int,inout username varchar(32))
begin
    set userId=2;
    set username='';
    select id,name into userId,username from users where id=userId;
end;
```

概括：
    1、可变变量INOUT:调用时可传入值，在调用过程中，可修改其值，同时也可返回值；
    2、INOUT参数集合了IN和OUT类型的参数功能；
    3、INOUT调用时传入的是变量，而不是常量；
### 存储过程条件语句
```
-- if判断语句
    if() then...
    elseif() then...
    else ...
    end if;
-- while语句
    while(表达式) do 
       ......  
	end while;
-- repeat语句
	repeat...until...end repeat;
    until判断返回逻辑真或者假，表达式可以是任意返回真或者假的表达式，只有当until语句为真是，循环结束
-- case语句
	case ...
    when ... then....
    when.... then....
    else ... 
    end case;
    当when内部的条件被case满足时，调用then后面的语句，都不满足则调用else。用法类似于java中的switch。
```
### 存储过程游标的使用
 游标是保存查询结果的临时区域
 示例：
 编写存储过程，使用游标，把users表中 id为偶数的记录逐一更新用户名
 ```
create procedure test11()
begin
    declare stopflag int default 0;
    declare username VARCHAR(32);
    -- 创建一个游标变量，declare 变量名 cursor ...
    declare username_cur cursor for select name from users where id%2=0;
    -- 游标是保存查询结果的临时区域
    -- 游标变量username_cur保存了查询的临时结果，实际上就是结果集
    -- 当游标变量中保存的结果都查询一遍(遍历),到达结尾，将变量stopflag设置为1，用于循环中判断是否结束
    declare continue handler for not found set stopflag=1;

    open username_cur; -- 打卡游标
    fetch username_cur into username; -- 游标向前走一步，取出一条记录放到变量username中
    while(stopflag=0) do -- 如果游标还没有结尾，就继续
        begin 
            -- 在用户名前门拼接 '_cur' 字符串
            update users set name=CONCAT(username,'_cur') where name=username;
            fetch username_cur into username;
        end;
    end while; -- 结束循环
    close username_cur; -- 关闭游标
end;
 ```
**注：存储过程转自[存储过程的使用](https://blog.csdn.net/qq_33157666/article/details/87877246**)


## mysql内置函数
### 数值函数
```
abs(x)          -- 绝对值 abs(-10.9) = 10
format(x, d)    -- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
ceil(x)         -- 向上取整 ceil(10.1) = 11
floor(x)        -- 向下取整 floor (10.1) = 10
round(x)        -- 四舍五入去整
mod(m, n)       -- m%n m mod n 求余 10%3=1
pi()            -- 获得圆周率
pow(m, n)       -- m^n
sqrt(x)         -- 算术平方根
rand()          -- 随机数
truncate(x, d)  -- 截取d位小数
```
###时间日期函数
```
now(), current_timestamp();     -- 当前日期时间
current_date();                 -- 当前日期
current_time();                 -- 当前时间
date('yyyy-mm-dd hh:ii:ss');    -- 获取日期部分
time('yyyy-mm-dd hh:ii:ss');    -- 获取时间部分
date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j'); -- 格式化时间
unix_timestamp();               -- 获得unix时间戳
from_unixtime();                -- 从时间戳获得时间
```
## 字符串函数
```
length(string)          -- string长度，字节
char_length(string)     -- string的字符个数
substring(str, position [,length])      -- 从str的position开始,取length个字符
replace(str ,search_str ,replace_str)   -- 在str中用replace_str替换search_str
instr(string ,substring)    -- 返回substring首次在string中出现的位置
concat(string [,...])   -- 连接字串
charset(str)            -- 返回字串字符集
lcase(string)           -- 转换成小写
left(string, length)    -- 从string2中的左边起取length个字符
load_file(file_name)    -- 从文件读取内容
locate(substring, string [,start_position]) -- 同instr,但可指定开始位置
lpad(string, length, pad)   -- 重复用pad加在string开头,直到字串长度为length
ltrim(string)           -- 去除前端空格
repeat(string, count)   -- 重复count次
rpad(string, length, pad)   --在str后用pad补充,直到长度为length
rtrim(string)           -- 去除后端空格
strcmp(string1 ,string2)    -- 逐字符比较两字串大小
```


### 聚合函数
```
count()
sum();
max();
min();
avg();
group_concat()
```
### 其他常用函数
```
md5();
default();
```
## 自定义函数
```
-- 新建
    CREATE FUNCTION function_name (参数列表) RETURNS 返回值类型
        函数体
    - 函数名，应该合法的标识符，并且不应该与已有的关键字冲突。
    - 一个函数应该属于某个数据库，可以使用db_name.funciton_name的形式执行当前函数所属数据库，否则为当前数据库。
    - 参数部分，由"参数名"和"参数类型"组成。多个参数用逗号隔开。
    - 函数体由多条可用的mysql语句，流程控制，变量声明等语句构成。
    - 多条语句应该使用 begin...end 语句块包含。
    - 一定要有 return 返回值语句。

-- 删除
    DROP FUNCTION [IF EXISTS] function_name;

-- 查看
    SHOW FUNCTION STATUS LIKE 'partten'
    SHOW CREATE FUNCTION function_name;

-- 修改
    ALTER FUNCTION function_name 函数选项
```
## 用户和权限管理
```
-- root密码重置
    1. 停止MySQL服务
    2.  [Linux] /usr/local/mysql/bin/safe_mysqld --skip-grant-tables &
        [Windows] mysqld --skip-grant-tables
    3. use mysql;
    4. UPDATE `user` SET PASSWORD=PASSWORD("密码") WHERE `user` = "root";
    5. FLUSH PRIVILEGES;
    用户信息表：mysql.user
-- 刷新权限
    FLUSH PRIVILEGES;
-- 增加用户
	CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
    - 必须拥有mysql数据库的全局CREATE USER权限，或拥有INSERT权限。
    - 只能创建用户，不能赋予权限。
    - 用户名，注意引号：如 'user_name'@'192.168.1.1'
    - 密码也需引号，纯数字密码也要加引号
    - 要在纯文本中指定密码，需忽略PASSWORD关键词。要把密码指定为由PASSWORD()函数返回的混编值，需包含关键字PASSWORD

-- 重命名用户
	RENAME USER old_user TO new_user

-- 设置密码
    SET PASSWORD = PASSWORD('密码')  -- 为当前用户设置密码
    SET PASSWORD FOR 用户名 = PASSWORD('密码') -- 为指定用户设置密码

-- 删除用户
	DROP USER 用户名

-- 分配权限/添加用户
	GRANT 权限列表 ON 表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
    - all privileges 表示所有权限
    - *.* 表示所有库的所有表
    - 库名.表名 表示某库下面的某表
    GRANT ALL PRIVILEGES ON `pms`.* TO 'pms'@'%' IDENTIFIED BY 'pms0817';

-- 查看权限
	SHOW GRANTS FOR 用户名
-- 查看当前用户权限
    SHOW GRANTS; 或 SHOW GRANTS FOR CURRENT_USER; 或 SHOW GRANTS FOR CURRENT_USER();

-- 撤消权限
    REVOKE 权限列表 ON 表名 FROM 用户名
    REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名   -- 撤销所有权限

-- 权限层级
	要使用GRANT或REVOKE，您必须拥有GRANT OPTION权限，并且您必须用于您正在授予或撤销的权限。
全局层级：全局权限适用于一个给定服务器中的所有数据库，mysql.user
    GRANT ALL ON *.*和 REVOKE ALL ON *.*只授予和撤销全局权限。
数据库层级：数据库权限适用于一个给定数据库中的所有目标，mysql.db, mysql.host
    GRANT ALL ON db_name.*和REVOKE ALL ON db_name.*只授予和撤销数据库权限。
表层级：表权限适用于一个给定表中的所有列，mysql.talbes_priv
    GRANT ALL ON db_name.tbl_name和REVOKE ALL ON db_name.tbl_name只授予和撤销表权限。
列层级：列权限适用于一个给定表中的单一列，mysql.columns_priv
    当使用REVOKE时，您必须指定与被授权列相同的列。

-- 权限列表
    ALL [PRIVILEGES]    -- 设置除GRANT OPTION之外的所有简单权限
    ALTER   -- 允许使用ALTER TABLE
    ALTER ROUTINE   -- 更改或取消已存储的子程序
    CREATE  -- 允许使用CREATE TABLE
    CREATE ROUTINE  -- 创建已存储的子程序
    CREATE TEMPORARY TABLES     -- 允许使用CREATE TEMPORARY TABLE
    CREATE USER     -- 允许使用CREATE USER, DROP USER, RENAME USER和REVOKE ALL PRIVILEGES。
    CREATE VIEW     -- 允许使用CREATE VIEW
    DELETE  -- 允许使用DELETE
    DROP    -- 允许使用DROP TABLE
    EXECUTE     -- 允许用户运行已存储的子程序
    FILE    -- 允许使用SELECT...INTO OUTFILE和LOAD DATA INFILE
    INDEX   -- 允许使用CREATE INDEX和DROP INDEX
    INSERT  -- 允许使用INSERT
    LOCK TABLES     -- 允许对您拥有SELECT权限的表使用LOCK TABLES
    PROCESS     -- 允许使用SHOW FULL PROCESSLIST
    REFERENCES  -- 未被实施
    RELOAD  -- 允许使用FLUSH
    REPLICATION CLIENT  -- 允许用户询问从属服务器或主服务器的地址
    REPLICATION SLAVE   -- 用于复制型从属服务器（从主服务器中读取二进制日志事件）
    SELECT  -- 允许使用SELECT
    SHOW DATABASES  -- 显示所有数据库
    SHOW VIEW   -- 允许使用SHOW CREATE VIEW
    SHUTDOWN    -- 允许使用mysqladmin shutdown
    SUPER   -- 允许使用CHANGE MASTER, KILL, PURGE MASTER LOGS和SET GLOBAL语句，mysqladmin debug命令；允许您连接（一次），即使已达到max_connections。
    UPDATE  -- 允许使用UPDATE
    USAGE   -- “无权限”的同义词
    GRANT OPTION    -- 允许授予权限

```
## 表维护
```
-- 分析和存储表的关键字分布
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE 表名 

-- 检查一个或多个表是否有错误
CHECK TABLE tbl_name [, tbl_name] ... [option] 
option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}

-- 整理数据文件的碎片
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name]

```