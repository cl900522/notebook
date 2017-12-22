Python操作Mysql
==========

## 安装模块
安装MySQLdb，请访问 http://sourceforge.net/projects/mysql-python ，(Linux平台可以访问：https://pypi.python.org/pypi/MySQL-python) 从这里可选择适合您的平台的安装包，分为预编译的二进制文件和源代码安装包。
如果您选择二进制文件发行版本的话，安装过程基本安装提示即可完成。如果从源代码进行安装的话，则需要切换到MySQLdb发行版本的顶级目录，并键入下列命令:

```shell
$ gunzip MySQL-python-1.2.2.tar.gz
$ tar -xvf MySQL-python-1.2.2.tar
$ cd MySQL-python-1.2.2
$ python setup.py build
$ python setup.py install
```

## 示例代码
### 数据查询
```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import MySQLdb

# 打开数据库连接
db = MySQLdb.connect("localhost","testuser","test123","TESTDB" )

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# 使用execute方法执行SQL语句
cursor.execute("SELECT VERSION()")

# 使用 fetchone() 方法获取一条数据
data = cursor.fetchone()

print "Database version : %s " % data

# 关闭数据库连接
db.close()
```

Python查询Mysql使用 fetchone() 方法获取单条数据, 使用fetchall() 方法获取多条数据。
* fetchone(): 该方法获取下一个查询结果集。结果集是一个对象
* fetchall():接收全部的返回结果行.
* rowcount: 这是一个只读属性，并返回执行execute()方法后影响的行数。

### 修改数据
```python
import MySQLdb

# 打开数据库连接
db = MySQLdb.connect("localhost", "testuser", "test123", "TESTDB")

# 使用cursor()方法获取操作游标
cursor = db.cursor()

# SQL 插入语句
sql = "INSERT INTO EMPLOYEE(FIRST_NAME, \
       LAST_NAME, AGE, SEX, INCOME) \
       VALUES ('%s', '%s', '%d', '%c', '%d' )" % \
       ('Mac', 'Mohan', 20, 'M', 2000)
try:
   # 执行sql语句
   cursor.execute(sql)
   # 提交到数据库执行
   db.commit()
except:
   # 发生错误时回滚
   db.rollback()

# 关闭数据库连接
db.close()
```

## 错误处理
DB API中定义了一些数据库操作的错误及异常，下表列出了这些错误和异常:

异常 |描述
--|--
Warning |当有严重警告时触发，例如插入数据是被截断等等。必须是 StandardError 的子类。
Error |警告以外所有其他错误类。必须是 StandardError 的子类。
InterfaceError |当有数据库接口模块本身的错误（而不是数据库的错误）发生时触发。 必须是Error的子类。
DatabaseError |和数据库有关的错误发生时触发。 必须是Error的子类。
DataError |当有数据处理时的错误发生时触发，例如：除零错误，数据超范围等等。 必须是DatabaseError的子类。
OperationalError |指非用户控制的，而是操作数据库时发生的错误。例如：连接意外断开、 数据库名未找到、事务处理失败、内存分配错误等等操作数据库是发生的错误。 必须是DatabaseError的子类。
IntegrityError |完整性相关的错误，例如外键检查失败等。必须是DatabaseError子类。
InternalError |数据库的内部错误，例如游标（cursor）失效了、事务同步失败等等。 必须是DatabaseError子类。
ProgrammingError |程序错误，例如数据表（table）没找到或已存在、SQL语句语法错误、 参数数量错误等等。必须是DatabaseError的子类。
NotSupportedError |不支持错误，指使用了数据库不支持的函数或API等。例如在连接对象上 使用.rollback()函数，然而数据库并不支持事务或者事务已关闭。 必须是DatabaseError的子类。

## 常见问题
1. 在安装MySQL-python-1.2.3.win-amd64-py2.7.exe时，提示：Python version 2.7 required，which was not found in the registry

>请检查并确保操作系统、python和模块都使用32bit或者64bit，否则造成注册表无法发现
>MySQLdb for python（32/64位）下载地址：http://www.codegood.com/archives/129
