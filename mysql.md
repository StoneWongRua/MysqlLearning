数据库环境搭建学习，建议学习5.6以上版本

#### 常用的SQL命令，授权管理学习

##### 创建数据库

```
create database db_name;
```

##### 创建表格

```
CREATE TABLE IF NOT EXISTS `db_table`(
	`id` INT UNSIGNED AUTO_INCREMENT,
	`name` VARCHAR(100) NOT NULL,
	PRIMARY KEY ( `id` )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

##### 根据已有的表创建新表(使用旧表B创建新表A)

```
# 复制完整的字段结构和索引
create table table_new like table_old

# 复制字段结构但不包括索引
create table table_new as select col1,col2.. from table_old definition only

# 复制数据，但是会丢失索引
create table as select 
```

##### 删除数据库

###### delete from和truancate的联系与区别

1. 事务上的区别
   1. **delete** 语句在执行删除的过程中是逐行删除的，而且每次删除一行，都会在事务日志中为所删除的每行记录一项，产生rollback，在事务提交以后才会生效。所以可以回滚。
   2. truncate 则是通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放，而且操作立即生效，自动提交，原数据不会放到rollbac segment中，所以无法恢复。在删除的过程中不会激活与表相关的删除触发器，从而加快了执行速度。
2. 索引上的区别
   1. 当表被truncate后，这个表和索引所占的空间会恢复到初始大小。a
   2. delete操作不会减少表或者索引所占用的空间。
3. 应用范围的区别
   1. truncate只能对table使用。
   2. delete可以对table和view使用。

###### drop的理解使用

1. drop语句会将表的结构删除，还会删除被依赖的约束、触发器、索引。
2. 删除之后，依赖于该表的存储过程以及函数将保留，但是会变为invalid状态。
3. 可以删除表结构。

##### 修改数据库

###### 修改表名

```
rename table old_name to new_name
```

要注意的是，在执行rename的时候，该表不能被锁定或者活动的事务，而且还要拥有对原表的ALTER和DROP权限，以及对新表的CREATE和INSERT权限。

如果是多表更名，当mysql遇到错误的时候，所有更名的表都会回退到最初的状态。

###### 修改表中某一行的某个值

```
# 把姓名为张三的人的年龄改成18岁
UPDATE  t_user SET age=18 WHERE NAME='张三'
```

- update可以同时更新一个或多个字段。
- update可以在 WHERE 子句中指定任何条件。
- update可以在一个单独表中同时更新数据。

###### 添加，修改，删除表的列，约束等表的定义

统一使用ALTER TABLE命令。

对列修改示例：

```
# 在表A的id列后面添加a列
ALTER TABLE `A`
ADD COLUMN `a`  INT AFTER `id`;

# 修改列名a为b
alter table t_user change a b int;

# 删除表A中的a列
ALTER TABLE A DROP COLUMN a;
```

对约束增加删除示例：

```
# 添加非空约束

# 建表的时候添加
CREATE TABLE t_user(user_id INT(10) NOT NULL);

# 通过ALTER 语句添加
ALTER TABLE t_user MODIFY user_id INT(10) NOT NULL;
ALTER TABLE t_user CHANGE user_id user_id INT(10) NOT NULL;

# 添加唯一约束

# 建表时直接添加
CREATE TABGLE t_user(user_id INT(10) UNIQE);

# 通过ALTER语句添加
ALTER TABLE t_user MODIFY user_id INT(10) UNIQUE;
ALTER TABLE t_user CHANGE user_id user_id INT(10) UNIQUE;
ALTER TABLE t_user ADD UNIQUE(user_id);

#删除唯一性约束
ALTER TABLE t_user DROP INDEX user_id;

PRIMARY KEY(主键约束)

# 添加主键约束

# 建表时直接添加
CREATE TABLE t_user(user_id INT(10) PRIMARY KEY);

# 通过ALTER语句添加
ALTER TABLE t_user MODIFY user_id INT(10) PRIMARY KEY;
ALTER TABLE t_user CHANGE user_id user_id INT(10) PRIMARY KEY;
ALTER TABLE t_user ADD PRIMARY KEY(user_id);


# 删除主键约束
ALTER TABLE t_user DROP PRIMARY KEY;

需要注意的是，主键约束相当于(唯一约束+非空约束)。所以一张表中最多有一个主键约束,如果设置多个主键,就会出现如下提示：Multiple primary key defined!!!
而且在删除主键约束前，如果有自增长需要先删除自增长,如果不删除自增长就无法删除主键约束。

# 添加外键约束（其中对应的字段只能是主键或者唯一约束修饰的字段）
# 假设存在两张表，分别是student和class，其中student中存在标识班级的class_id，而且class也有主键class_id表示班级号，那么可以这样添加外键约束

ALTER TABLE students ADD CONSTRAINT FK_CLA_ID FROEIGN KEY(class_id) REFERENCES class(class_id);

# 外键中的级联关系有以下几种情况：
1. ON DELETE CASCADE 删除主表中的数据时，从表中的数据随之删除
2. ON UPDATE CASCADE 更新主表中的数据时，从表中的数据随之更新
3. ON DELETE SET NULL 删除主表中的数据时，从表中的数据置为空
4. 默认 删除主表中的数据前需先删除从表中的数据，否则主表数据不会被删除

# 删除外键约束
ALTER TABLE students DROP FOREIGN KEY FK_CLA_ID;

需要注意的是，在插入数据时，先插入主表中的数据，再插入从表中的数据。
		    而删除数据时，先删除从表中的数据，再删除主表中的数据。

```
