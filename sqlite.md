# sqlite3数据库
## 1. sqlite是什么
sqlite是一款轻型的数据库，是遵守ACID的关系型数据库管理系统，它包含在一个相对小的C库中。
## 2. 安装sqlite3:
```sh
sudo apt-get install sqlite3
```
![](http://p1.bpimg.com/1949/3a1a101bb0b946f0.png)
## 3. 安装sqlite的C语言库
```sh
1. sudo apt-get update
2. sudo apt-get install  libsqlite3-dev
检查是否安装成功： cd  /usr/include 中查看是否有sqlite3.h文件
```
## 4. sqlite基本命令
1. 建数据库：
```sql
   sqlite3 test.db    //注意sqlite的版本
```
2. 查看帮助：
```sql
   sqlite> .help
```
3. 文件存放位置：
```sql
   sqlite> .database
```
4. 退出：
```sql
   sqlite> .quit
```
5. 查看表：
```sql
   sqlite> .tables
```
6. 显示表的结构：
```sql
   sqlite> .schema
```
## 5. sqlite3常见语句
1. 创建表：
```sql
sqlite> create table usr(id integer primary key, name text, age integer null, gender text,
salary real not null);
```
2. 删除表
```sql
sqlite> drop table usr；
```
3. 插入数据：
```sql
sqlite> insert into usr(id, name, age, salary) values(2, ‘liu’, 20, 6000);
```
4. 删除数据:
```sql
sqlite> delete from usr where id = 2 and name = ‘lisi’;    // and(与）or（或）

sqlite> delete from usr where id = 2 or name = ‘lisi’;     // 删除一条记录 
```
5. 修改数据：
```sql
sqlite> update usr set gender = ‘man’，name = ‘lisi’ where id = 3;
```
6. 查询数据：
```sql
sqlite>.head on
sqlite>.mode column       
sqlite> select * from usr where id = 2;       //此三个命令被用来设置正确格式化的输出。
```
7. 修改表的名称
```sql
sqlite> alter table oldname rename to newname;
```
# 参考学习
* [RUNOOB.COM] (http://www.runoob.com/sqlite/sqlite-select.html)

