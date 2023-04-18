# 3 MySQL数据库

## 3.1 数据库相关指令

（1）创建数据库

```
//character是设置字符集，collate是设置校验规则
CREATE DATABASE db CHARACTER SET utf8 COLLATE utf8_bin;
```

（2）查看、删除数据库

```
show databases;
drop database db;
```

（3）备份数据库

```
//dos下执行
//备份整个数据库
mysqldump -u root -p -B db > D:\\back.sql;
//备份数据库的某个表
mysqldump -u root -p db table1 > D:\\back.sql;
```

（4）恢复数据库

```
//进入mysql命令行后执行
source D:\\back.sql
```

（5）创建表

```
create table worker(

			id int,

			name varchar(32),

			sex varchar(4));
```

## 3.2 Mysql常用数据类型     

1. 整性：int(四个字节)     
    （1）若没有指定unsinged,则是有符号的。       

  ```
  //指定无符号int
  create table t1(id int unsinged);
  ```

2. 小数：

  float(双精度)，double(双精度)，decimal[M,D]:共M位，小数位数D位        
  （1）M最大为65，D最大是30       
  （2）如果D被省略，默认是0。如果M被省略，默认是0         

3. 文本类型：char（0-255），varchar（0-65535），text（0-2^16-1）        
    （1)  char（n):固定长度字符串，最大255**字符**。n表示字符数。     
    （2）varchar（n）：可变长度字符串，最大65532**字节**（其中3个字节用于记录大小）。n表示字符。    
    （3）char是定长，varchar是变长即根据实际占用空间分配。varchar需要1-3个字节来记录长度。     
    （4）查询速度char>varchar,所以如果数据定长时推荐使用char。
    （5）text可以视为varchar列

4. 日期类型：datetime(年月日时分秒)，timestamp（时间戳）     

   ```
   //设置自动更新
   login_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTA
   ```
## 3.3 CRUD    

### 3.3.1 修改表

```
//添加列
alter table t1 add name varchar(32);
//修改列
alter table t1 modify name varchar(64) not null;
//删除列
alter table t1 drop name;
```

### 3.3.2 DML语句

1. 字符和日期类型都应该包含在单引号中。
2. 如果某个列没有指定not null，则添加数据时没有给定值会默认赋空，如果有默认值则赋默认值。
3. 给定默认值：使用default

```
insert into t1 values(1, 'apple');
update t1 set name = 'hello' where id = 1;
delete from t1 where id = 1;  
select * from t1;
```

### 3.3.3 where子句中常用的运算符

1. between……and……：左闭右闭

2. in（set）：在某一区间

3. like/not like：模糊查询 

   ```
   //%匹配任意字符串（字符串大小可为0）
   //_匹配任意字符
   select * from student where name like '%a%';
   ```

### 3.3.4 order by

1. asc代表升序（默认），desc代表降序

2. 可以设置多个排序

   ```
   select math from student order by math;
   ```

### 3.3.5 聚集函数

1. count(*):返回满足条件的记录的行数，不会排除null。
2. count（列）：统计满足条件的某列有多少个，但会排除为null的。
3. avg():求平均数
4. min()/max()：求最大最小值
5. sum()：求总和

tips：统计函数不能出现在where子句中。因为聚集函数要对全列数据进行计算，因而使用它的前提是：结果集已经确定！而where子句是对结果进行筛选。两者放在一起矛盾。   

### 3.3.6 group by

1. where只能用在group by前面
2. having子句用于对group by之后的过滤。

```
select avg(sal) from emp group by deptno having avg(sal) > 2000;
```

## 3.4 相关函数

### 3.4.1 字符串方法       

tips：里面的索引从1开始     
（1）charset(列名)：查看字符编码方式     
（2）concat（str1，str2，……）：拼接字符串，可以是列名。       
（3）instr（str，substr）：substr在str中出现的位置，没有返回0.       
（4）ucase（列名）：转换成大写。lcase（列名）：转换成小写            
（5）left（str，len）：从str左边取len个字符           
（6）right（str，len）：从str右边取len个字符          
（7）length（str）：统计str的大小，按字节返回       
（8）replace（str，str1,str2）:在str中用str2替换成str1（只替换查询结果）       
（9）strcmp（str1，str2）：比较两串大小     
（10）substring（str，pos，len）：从str的pos开始取len个字符。       
（11）trim（str）：取出前后两端的空格，ltrim去除左边的，rtrim去除右边的          

### 3.4.2 数学相关函数    

（1）abs（列名）：求绝对值    
（2）bin（列名）：十进制转二进制     
（3）ceiling（列名）：向上取整     
（4）conv（列名，进制1，进制2）：从进制1转成进制2       
（5）floor（列名）：向下取整            
（6）format（列名，n）：保留n位小数，规则为四舍五入            
（7）hex（列名）：转16进制        
（8）least（列名，列名，……）：求最小值     
（9）mod（列名，n）：模n          
（10）rand（n）：返回随机数，范围是0≤n≤1。如果添加n（可以不添加），返回固定随机数，n不变随机数不变。     

### 3.4.3 日期相关函数     

（1）current_date():当前日期     
（2）current_time（）:当前时间        
（3）current_timestamp()：当前时间戳，可以直接用到insert语句中            
（4）Date（datetime）：返回datetime的日期部分             
（5）Date_ADD(date, interval n minute/hour/second/year/day):加时间，date还可以是datetime，timestamp等。          
（7）Date_SUB(date， interval n minute/hour/second/year/day):减时间     
（8）Datediff（date1，date2）：两个日期差，结果是天。date1-date2                    
（6）now（）：返回当前时间，不会改变，时间戳每次可以更新。       
（7）Timediff（date1，date2）：两个时间差，结果是时分秒     
（8）year（），month（），day（）：只获取年/月/日     
（9）unix_timestamp():返回1970-1-1到现在的秒数     
（10）from_unixtime(n,'%Y-%m-%d'):将unix_timestamp的秒数转换成指定格式的日期。         
tips：from_unixtime(n,'%Y-%m-%d %H:%i:%s'):在开发中可以存放一个整数表示时间，通过这个函数进行转换。       

### 3.4.4 加密和系统函数        

（1）user（）：查看登录到mysql的用户以及登陆的ip      
（2）database（）：查询当前使用的数据库名称       
（3）MD5（str）：为str进行MD5加密。加密结果是32位的。    
（4）SHA1（str）：为str进行SHA加密       

### 3.4.5 流程控制函数       

（1）if（exp1,exp2,exp3）:如果exp1为真，则返回exp2的值，否则返回exp3的值。            
（2）ifnul（exp1,exp2）:如果exp1不为空则返回exp1，否则……      
（3）select case when exp1 then exp2 when exp3 then exp4 else exp5 end:如果exp1为真，返回exp2。如果exp3为真，返回exp4，否则返回exp5。类似于多分支语句。（不会返回多个值）          

```
SELECT CASE WHEN TRUE THEN 'jack' WHEN FALSE THEN 'tom' ELSE 'mary';
```

## 3.5 复杂表查询

1. 判断是否为空使用is（not）        

2. 日期类型可以直接比较

3. %表示0到多个字符，_表示单个任意字符。

4. 分页查询：

   ```
   //表示从start+1 行开始取，取出rows行，start从0开始计算  
   select …… limit start，rows    
   //分页公式
   select * from emp
   limit 每页显示的记录数*(第几页-1), 每页显示记录数
   ```

5. 顺序：from->where->group by->having->order by->limit    

6. 创建和emp表结构相同的表。    

   ```
   create table my_table02 like emp
   ```

7. 自连接

   ```
   //自连接必须给表取别名，select时也必须指明是哪张表
   select worker.ename as 'w', manager.ename as 'm'
   from emp as worker, emp as manager
   where worker.mgr = manager.empno;
   ```

8. 多列子查询：

   ```
   // (字段1,字段2) = (select 字段1, 字段2)
   ```

9. 多行子查询

   ```
   //使用min、max、all、any等函数
   //>all表示大于所有，>any表示大于其中一个
   ```

10. 左连接：

    左侧表完全显示，如果没有某字段显示为空

    ```
    //表1 left join 表2 on 连接条件
    ```

8. 右连接：

   ```
   //表1 right join 表2 on 连接条件   
   ```

9. 表复制

   ```
   //使用insert语句进行复制
   insert into t1 select * from t1;
   ```

### 3.5.1 合并/交集查询

1. union：取出结果的并集，自动去除重复行
2. intersect：取出结果的交集，自动去除重复哈那个
3. 如果需要保留重复行，使用union/intersect all

## 3.6 mysql约束

1. primary key：主键，表示某列的值不能重复（可以唯一标识表），只能有一个主键       
（1）主键不能为null        
（2）复合主键：primary key（column1，column2，……）      
2. not null：非空约束     
3. unique:该列值不能重复     
（1）可以有多个null     
（2）一张表可以有多个unique字段      
4. foreign key：      
（1）foreign key（本表字段）reference 主表（字段）；     
（2）外键指向的字段必须是主键或者unique字段         
（3）表的类型必须是innodb，才支持外键     
（4）外键和主键的字段的类型必须一致，长度可以不同     
（5)   外键字段的值必须在主表的字段中出现或者为null     
（6）主外键的关系一旦建立，就不能随意删除主表数据      
5. check约束      
（1）在字段后强制添加约束          
## 3.7 自增长
1.  格式

   ```
   //字段名 int primary key auto_increment；
   id int primary key auto_increment;
   ```

2. 在进行insert操作时，该字段可以直接填null。   

3. 自增长一般和主键配合使用，但也可以和unique配合使用。

4. 自增长默认从1开始     

  ```
  //alter table 表名 auto_increment = 新值；    
  ALTER TABLE t1 AUTO_INCREMENT = 1;
  ```

5. 如果添加数据时，给自增长字段指定了值，则以指定的值为准。    
## 3.8 索引        

```
//create index 索引名 on 表名（字段）;
create index empno_index on emp(empno);
```

创建索引后，只对创建了索引的字段有效。创建索引后，文件会变大，因为索引本身也会占用空间。    

### 3.8.1 索引的原理    
1. 没有索引时，进行的是全表扫描，查询速度慢。   

2. 当创建索引后，会创建该字段的数据结构，比如二叉树等

3. 索引的代价：

   （1）磁盘占用（索引本身会占用空间）；

   （2）对update/delete/insert语句的效率有一定的影响。（因为dml语句的执行必须修改该字段的数据结构）   
### 3.8.2 索引的类型
1. 主键索引：主键自动成为主索引。        
    创建表后添加主键索引的方式：      

  ```
  alter table t2 add primary key (id);    
  ```

2. 唯一索引（unique）：unique的字段自动成为唯一索引          
    创建表后添加唯一索引的方式：     

  ```
  create unique index id_index on t1(id);    
  ```

3. 普通索引（index）：       
    创建表后添加普通索引：    

  ```
  create index name_index on t1(name);     
  alter table t1 add index name_index(name)； 
  ```

4. 全文索引（fulltext）:

  开发中一般不适用mysql自带的全文索引，考虑使用全文索引Solr和ElasticSearch
  tips：如果某列的值是不会重复的，优先使用unique索引，否则使用普通索引。     
### 3.8.3 删除索引
1. 删除普通/unique索引

   ```
   //drop index 索引名 on 表名;
   drop index id_index on t1;
   ```

2. 删除主键索引：      

  ```
  //alter table 表名 drop primary key; 
  alter table t1 drop primary key;
  ```
### 3.8.4 查看索引
```
//show index from 表名;
//show indexes from 表名;
//show keys from 表名;
//desc 表名;    
```

### 3.8.5 何时使用索引
1. 较频繁地作为查询条件的字段应该创建索引
2. 唯一性太差的字段不太适合创建索引
3. 更新频繁（经常执行dml语句的字段）的字段不适合创建索引        
## 3.9 事务    
### 3.9.1 事务概念
1. 事务用于保证数据的一致性，由一组dml语句组成（update/insert/delete）。该组语句要么全部成功，要不全部失败
2. 当执行事务操作时，mysql会在表上加锁，防止其他用户更改表的数据

### 3.9.2 事务操作
1. start transaction/set autocommit = off:开始一个事务
2. savepoint 保存点名：设置保存点
3. rollback to 保存点名：回退到某保存点
4. rollback：回退到事务开始的状态
5. commit：提交事务，所有操作生效，不能回退，该事务所定义的所有保存点都会被删除，并会释放锁。    
### 3.9.3 事务细节
1. 如果不开始事务，默认情况下，mysql的dml操作是自动提交的，不能回滚。   
2. mysql的事务机制需要innodb地存储引擎才可以使用，myisam存储引擎不能使用。     
### 3.9.4 事务的隔离级别
1. 脏读：A事务读取B事务尚未提交的数据，此时如果B事务发生错误并执行回滚操作，那么A事务读取到的数据就是脏数据。    
    tips：读取到的数据被回滚了，所以该数据是脏数据。    

2. 不可重复读：事务A读取了某数据，事务B修改（此时强调update操作）了该数据并提交事务，此时A再次读该数据，两次数据**内容**不一致。  
    tips：读取到了被其他事务修改过的数据（其他事务已经commit）    

3. 幻读：事务A读取了某些数据，事务B修改（此时强调insert和update操作）了这些数据并提交事务，此时A再次读该数据，两次数据查询结果数量不同（先是2条结果，后是三条结果）

   tips：前后多次读取，数据总量不一致     
### 3.9.5 四种隔离级别
1. 读未提交（read uncommitted）：脏读+不可重复读+幻读（不加锁）
2. 读已提交（read committed）：不可重复读+幻读（不加锁）
3. 可重复读（repeatable uncommitted）：幻读（不加锁）
4. 可串行化（serializable）：（加锁）     
### 3.9.6 隔离级别操作
1. select @@transaction_isolation：查看当前会话隔离级别
2. select @@global.transaction_isolation:查看系统当前隔离级别。    
3. set session transaction isolation level [隔离级别]：设置隔离级别
4. mysql默认的事务隔离级别是repeatable read
### 3.9.7 ACID特性
1. 原子性：事务不可分割
2. 一致性
3. 隔离性：多个并发事务之间要相互隔离
4. 持久性：事务一旦提交，对数据的改变就是永久性的。    
## 3.10 存储引擎
### 3.10.1 存储引擎概述

1. mysql的表类型由存储引擎决定，主要包括MyISAM，innoDB，Memory。    
2. 分类：事务安全型，非事务安全型
3. innoDB：支持事务、支持外键、支持行级锁
4. myisam：添加速度快，不支持外键和事务，支持表级锁
5. memory：数据存储在内存中（关闭mysql服务，数据会丢失，但表结构还在），执行速度很快（没有IO读写），默认支持索引（hash表）。       
### 3.10.2 如何选择存储引擎

1. 不需要事务处理：myisam
2. 需要事务：innoDB
3. 特殊地：memory（比如记录用户的在线状态）

## 3.11 视图

1. 概念：视图是一个虚拟表，其内容由查询定义，其数据来自对应的真实表（基表）。

   tips：视图并不是真实的表    

2. 通过视图可以修改基表的数据,基表的改变也会影响视图。

3. 视图可以有多个基表。     

4. 视图中可以再使用视图
### 3.11.1 视图的使用 
1. 创建视图：     

  ```
  create view emp_view01 as select empno,ename,job,deptno from emp;  
  ```

2. 更新新的视图：     

  ```
  alter view emp_view01 as select empno,ename,job from emp;  
  ```

3. 删除视图:     

  ```
  drop view emp_view01;     
  ```
### 3.11.2 视图的实践
1. 安全：只保留一部分可以公开的字段
2. 性能：join效率较低，使用视图将相关字段组合效率高呢广告
3. 灵活：
## 3.12 mysql管理

1. mysql中的用户都存储在系统数据库mysql里的user表中。

2. 做项目开发时，可以根据不同的人员，赋予不同的Mysql操作权限。   

3. 创建用户：    

  ```
  //create user 用户名@允许登陆位置 identified by 密码
  //用户名与登陆位置之间没有空格
  create user 'zuzu'@'localhost' identified by '123456';   
  ```

4. 删除用户：      

  ```
  drop user 'zuzu'@'localhost';
  ```

5. 不同的数据库用户，登录到DBMS后，根据相应的权限，可以操作的数据库和数据对象（表、视图、触发器）不一样。    

6. 修改密码：      

  ```
  //set password for '用户名'@'允许登陆位置'='新密码';
  set password for 'zuzu'@'localhost'='abcdef';  
  ```
### 3.12.1 Mysql权限

1. 给用户授权：        

  ```
  //grant 权限列表 on 库.对象名 to '用户名'@'登陆位置' [identified by '密码'];
  //密码可写可不写
  grant select,insert on testdb.news to 'zuzu'@'localhost';    
  ```

  （1）多个权限用逗号分隔          
  （2）库.对象名：“.”表示所有数据库的所有对象       
  （3）identified by可以省略也可以不写。如果该用户存在则会修改该用户密码。如果该用户不存在，则创建该用户。      
  （4）如果是所有权限：权限列表处写“all”       

2. 回收权限：     

  ```
  //revoke 权限列表 on 库.对象名 from '用户名'@'登陆位置';
  revoke insert on testdb.news from 'zxy'@'localhost';        
  ```
### 3.12.2 用户管理的细节

1. 在创建用户时如果不指定Host，则为%,%表示所有IP都有连接权限。      

   ```
   create user `jack` identified by '123456';      
   ```

2. 'zuzu'@'192.168.1.%':可以用%表示IP前面为192.168.1的都可以登录。

3. 在删除用户时，如果host不是%，需要明确指定host的值。      