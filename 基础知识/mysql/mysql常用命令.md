# 一、SQL语句分类:
* 1、DQL(Data Query Language)        数据库查询语言,select
* 2、DML(Data Manipulation Language)        数据库操作语言,insert、delete、update
* 3、DDL(Data Denifition Language)        数据库定义语言,create、drop、alter
* 4、TCL(Trasactional Control Languag)        事务控制语言,commit、rollback
* 5、DCL(Data Control Language)        数据控制语言用来授予或回收访问数据库的某种特权
* **sql语法:** select ... from ... where ... group by ... order by ...;
* **执行顺序:** from->where->group by->select->order by
# 二、常用命令
* 启动MYSQL:net start mysql;
* 关闭MYSQL:net start mysql;
* 连接MYSQL:mysql -u root -p;
* 退出MYSQL:exit;
* 查看MYSQL中的数据库:show databases;
* 使用指定数据库:use 数据库名;
* 创建数据库:create database 数据库名;
* 执行sql脚本:source 文件路径;
* 查看表中所有数据:select * from 表名;
* 查看表结构:desc 表名;desc为describe的缩写
# 三、排序
* select * from 表名 order by 字段 [asc|desc];(asc升序,desc降序)
# 四、函数
* ifnull函数用法:        ifnull(数据,数据为null时的值)
* distinct去重:         select distinct job from emp;
* union:                合并查询结果集
# 五、查询★★★
* 查询多个字段:             select 字段名,字段名,... from 表名;
* 条件查询:                 select 字段名,字段名,... from 表名 where 条件;
* 模糊查询: %可以匹配任意个字符, _可以匹配一个字符
* 内连接查询:select 表1.字段,...,表2.字段,..., from 表1 join 表2 on 条件;
* 左外连接查询(join左边的表1为主表,将主表中数据全部查出):select 表1.字段,...,表2.字段,..., from 表1 left join 表2 on 条件;
* 右外连接查询(join右边的表2为主表,将主表中数据全部查出):select 表1.字段,...,表2.字段,..., from 表1 right join 表2 on 条件;
# 六、表
* 表的创建:CREATE TABLE  [IF NOT EXISTS] `表名` (
	`字段名` 列类型 [属性] [索引] [注释]，
	`字段名` 列类型 [属性] [索引] [注释]，
	.......
	`字段名` 列类型 [属性] [索引] [注释]
) [表类型] [字符集设置] [注释]
* 表的删除:drop table if exists 表名;
* 表中插入数据:使用Inset into 表名(字段名) values(值);
# 七、视图
* 对视图对象的增删改查,会导致原表被操作
* create view 视图名 as select * from 表名;