# sqlite3的C语言接口的基本使用
```sh
注意：使用sqlite库进行代码的编译的时候，必须添加-lsqlite

例如:gcc xxx -lsqlite
```
## 1. 打开或者创建数据库:
```c
int sqlite3_open(char *path, sqlite3 **db);
 db:指向sqlite句柄的指针；
 成功返回0；
 失败返回错误码；
```
## 2. 出错判断:
```c
char *sqlite3_errmsg(sqlite3*);
```
## 3. 关闭数据库：
```c
int sqlite3_close(sqlite3 *db);
```
## 4. 执行sql操作：
```c
int sqlite3_exec(
  sqlite3*,                                  /* 数据库句柄 */
  const char *sql,                           /* SQL语句 */
  int (*callback)(void*,int,char**,char**),  /* 回调函数 */
  void *,                                    /* 函数参数 */
  char **errmsg                              /* 错误信息指针的地址 */
);
返回值： 成功返回0;
```
## 5. 利用回调函数查询数据库
```c
回调执行函数：每找到一条自动执行一次函数
typedef int （*sqlite3_callback）(
			void *para, 	//传递给函数的参数
			int f_num, 		//记录中包含的字段的数目
			char **f_value, //包含每个字段值的指针数组
			char **f_name	//包含每个字段名称的指针数组
			);
f_num 是查询到的符合条件的字段数(如果只查询一个字段的值，那么该值就是1)`
f_value是查询到的符合条件的一行记录的值(如果符合条件的字段数为1，那么该值就对应一个字段，而且只有一个值)
```
## 6. 程序示例
```c
#include <stdio.h>
#include <string.h>
#include <sqlite3.h>
#include <stdlib.h>

#define M 32
#define N 128

int do_insert(sqlite3 *db);
int do_delete(sqlite3 *db);
int do_show(sqlite3 *db);
int call_back(void *param, int column, char **value, char **name);
int do_show_1(sqlite3 *db);
int do_update(sqlite3 *db);

int main(int argc, char *argv[])
{
	sqlite3 *db;
	char *errmsg;
	char clean[M];
	int cmd;

	if (sqlite3_open("./my.db", &db) != SQLITE_OK)
	{
		printf("%s\n", sqlite3_errmsg(db));
		return -1;
	}

	int ret;
	//sql语句在编写时,如果需要分行写,那么需要添加连接符'\',而且连接符的后边不要带有空格
	if ((ret = sqlite3_exec(db, "create table stu(id integer, name vchar(32) not null,\
		score integer not null)", NULL, NULL, &errmsg)) != SQLITE_OK)
	{
		if (ret != 1)
		{
			printf("%s\n", errmsg);
			sqlite3_close(db);
			return -1;
		}
	}

	while (1)
	{
		printf("\e[32m*** 1:insert  2:delete  3:show  4:update  5:quit ***\e[0m\n");
		printf("please input your comd > ");

		if (scanf("%d", &cmd) != 1)
		{
			puts("input error");
			fgets(clean, sizeof(clean), stdin);
			continue;
		}

		switch (cmd)
		{
		case 1:
			do_insert(db);
			break;
		case 2:
			do_delete(db);
			break;
		case 3:
			do_show(db);
			//do_show_1(db);
			break;
		case 4:
			do_update(db);
			break;
		case 5:
			goto RET;
		default:
			break;
		}
	}

RET:
	sqlite3_close(db);
	return 0;
}

int do_insert(sqlite3 *db)
{
	int id;
	char name[M] = {0};
	int score;
	char sql[N] = {0};
	char *errmsg;

	printf("input  student id > ");
	scanf("%d", &id);

	printf("input  student name > ");
	scanf("%s", name);

	printf("input  student score > ");
	scanf("%d", &score);

	sprintf(sql, "insert into stu values(%d, '%s', %d)", id, name, score);
	if (sqlite3_exec(db, sql, NULL, NULL, &errmsg) != SQLITE_OK)
	{
		printf("%s\n", errmsg);
		return -1;
	}

	printf("\e[32minsert OK !\e[0m\n");
	return 0;
}

int do_delete(sqlite3 *db)
{
	int id;
	char sql[N] = {0};
	char *errmsg;

	printf("please input student id > ");
	scanf("%d", &id);

	sprintf(sql, "delete from stu where id = %d", id);

	if (sqlite3_exec(db, sql, NULL, NULL, &errmsg) != SQLITE_OK)
	{
		printf("%s\n", errmsg);
		return -1;
	}

	printf("\e[32mdelete OK !\e[0m\n");
	return 0;
}

int do_show(sqlite3 *db)
{
	char *errmsg;
	int a = 0;

	if (sqlite3_exec(db, "select * from stu", call_back, &a, &errmsg ) != SQLITE_OK)
	{
		printf("%s\n", errmsg);
		return -1;
	}

	printf("\e[32mshow OK !\e[0m\n");
	return 0;
}

int call_back(void *param, int column, char **value, char **name)
{
	int a;
	int i = 0;
	a = (*(int*)param)++;

	if ( a == 0)
	{
		for (i = 0; i < column; i++)
		{
			printf("%-15s", *(name++));
		}

		putchar(10);
	}

	for (i = 0; i < column; i++)
	{
		printf("%-15s", *(value++));
	}

	putchar(10);
	return 0;
}

int do_show_1(sqlite3 *db)
{
	char *errmsg;
	char **result, **temp;
	int nrow;
	int ncolumn;
	int i, j;

	if (sqlite3_get_table(db, "select * from stu", &result, &nrow, &ncolumn, &errmsg) != SQLITE_OK)
	{
		printf("%s\n", errmsg);
		return -1;
	}
	else
	{
		temp = result;
		for (i = 0; i < nrow + 1; i++)
		{
			for (j = 0; j < ncolumn; j++)
				printf("%-15s", *(temp++));
			putchar(10);
		}
	}

	sqlite3_free_table(result);
	printf("\e[32mshow OK !\e[0m\n");

	return 0;
}

int do_update(sqlite3 *db)
{
	int id;
	int score;
	char sql[N] = {0};
	char *errmsg;

	printf("please input student id >> ");
	scanf("%d", &id);

	printf("please input student score >> ");
	scanf("%d", &score);

	sprintf(sql, "update stu set score = %d  where id = %d", score, id);

	if (sqlite3_exec(db, sql, NULL, NULL, &errmsg) != SQLITE_OK)
	{
		printf("%s\n", errmsg);
		return -1;
	}

	puts("update OK !");
	return 0;
}
```
# 参考资料
* [网址] (http://www.runoob.com/sqlite/sqlite-c-cpp.html)
