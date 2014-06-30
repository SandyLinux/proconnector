9.13 错误和异常
==================

mysql.connector.errors模块为错误和被MySQL Connector/Python 抛出的警告定义了异常类。当你导入mysql.connector时，在此模块中定义的大多数类可用。

在此模块中定义的异常类大多遵循Python Database API Specification v2.0（PEP 249）。对于某些MySQL客户端或者服务器错误，抛出哪些异常并不是总是很清楚。讨论一个错误是否应该被一个打开的bug报告重新分类是件好事。

MySQL服务器的错误是根据他们的SQLSTATE值映射到Python异常（查看[Server Error Codes and Messages](http://dev.mysql.com/doc/refman/5.6/en/error-messages-server.html)）。下面的表显示SQLSTATE分类与Connector/Python抛出异常。然而，可以为每个服务器错误重新定义抛出哪个异常。注意,默认的异常是DatabaseError。

表9.1

	 +----------------+----------------------------+
	 | SQLSTATE Class | Connector/Python Exception |
	 +----------------+----------------------------+
	 |       02       | DataError                  |
	 +----------------+----------------------------+
	 |       02       | DataError                  |
	 +----------------+----------------------------+
	 |       07       | DatabaseError              |
	 +----------------+----------------------------+
	 |       08       | OperationalError           |
	 +----------------+----------------------------+
	 |       0A       | NotSupportedError          |
	 +----------------+----------------------------+
	 |       21       | DataError                  |
	 +----------------+----------------------------+
	 |       22       | DataError                  |
	 +----------------+----------------------------+
	 |       23       | IntegrityError             |
	 +----------------+----------------------------+
	 |       24       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       25       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       26       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       27       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       28       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       2A       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       2B       | DatabaseError              |
	 +----------------+----------------------------+
	 |       2C       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       2D       | DatabaseError              |
	 +----------------+----------------------------+
	 |       2E       | DatabaseError              |
	 +----------------+----------------------------+
	 |       33       | DatabaseError              |
	 +----------------+----------------------------+
	 |       34       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       35       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       37       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       3C       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       3D       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       3F       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       40       | InternalError              |
	 +----------------+----------------------------+
	 |       42       | ProgrammingError           |
	 +----------------+----------------------------+
	 |       44       | InternalError              |
	 +----------------+----------------------------+
	 |       HZ       | OperationalError           |
	 +----------------+----------------------------+
	 |       XA       | IntegrityError             |
	 +----------------+----------------------------+
	 |       0K       | OperationalError           |
	 +----------------+----------------------------+
	 |       HY       | DatabaseError              |
	 +----------------+----------------------------+

## 9.13.1 模块 errorcode

该模块包含MySQL服务器和客户端错误代码，被定义为以错误号为值的模块属性。使用错误代码代替错误号能使得源代码会易读些。

	 >>>from mysql.connector import errorcode
	 >>>errorcode.ER_BAD_TABLE_ERROR
	 1051

查看[Server Error Codes and Messages ](http://dev.mysql.com/doc/refman/5.6/en/error-messages-server.html)和 [Client Error Codes and Messages](http://dev.mysql.com/doc/refman/5.6/en/error-messages-client.html)。


## 9.13.2 异常 errors.Error

该异常是所有在errors模块内其他异常的基类。它能用于当except statement中抓取所有错误。

下面的例子显示我们怎么抓取语法错误：

	 import mysql.connector
	 try:
	  	 cnx = mysql.connector.connect(user='scott',database='employees')
		 cursor = cnx.cursor()
		 cursor.execute("SELECT * FORM employees") #查询语法错误
		 cnx.close()
	 except mysql.connector.Error as err:
		 print("Something went wrong: {}".format(err))

初始化异常支持几个可选的参数，亦即是msg,errno,values和sqlstate。他们所有都是可选的并且默认为None。errors.Error被Connector/Python内部用于抛出MySQL 客户端和服务器错误，不应被用于你的应用来抛异常。
	
下面的例子显示当不使用参数或者使用一组参数时的结果：
	
	 >>> from mysql.connector.errors import Error
	 >>> str(Error())
	 'Unknown error'

	 >>> str(Error("Oops! There was an error."))
	 'Oops! There was an error.'

	 >>> str(Error(errno=2006))
	 '2006: MySQL server has gone away'

	 >>> str(Error(errno=2002, values=('/tmp/mysql.sock', 2)))
	 "2002: Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)"

	 >>> str(Error(errno=1146, sqlstate='42S02', msg="Table 'test.spam' doesn't exist"))
	 "1146 (42S02): Table 'test.spam' doesn't exist"

使用错误号为1146的例子用于Connector/Python接收到MySQL 服务器一个错误的包的时候。信息被解析过并被传递给Error异常，如图所示。

每个Error 异常的子类能用前面提到的参数初始化。此外，每个实例含有errno,msg和sqlstate属性，可以用于你的代码中。

下面的例子显示怎么处理当删除一个不存在的表（DROP TABLE 语句中不包含IF EXISTS引起的）的错误：
	
	 import mysql.connector
	 from mysql.connector import errorcode
	 cnx = mysql.connector.connect(user='scott', database='test')
	 try:
	  	 cur.execute("DROP TABLE spam")
	 except mysql.connector.Error as err:
		 if err.errno == errorcode.ER_BAD_TABLE_ERROR:
			 print("Creating table spam")
		 else:
		  raise

在Connector/Python 1.1.1之前，传递给errors.Error()的原信息以这样的方式不会保存，它可以被检索。相反，Error.msg属性被带错误号和SQLSTATE 值格式化。随着1.1.1，仅原信息被保存在Error.msg属性中。格式化的值连同错误号和SQLSTATE 值可以通过打印或者错误对象的字符串表现形式来获得。例如：

	 try:
		 conn = mysql.connector.connect(database='baddb')
	 except mysql.connector.Error as e:
		 print "Error code:",e.errno
		 print "SQLSTATE value:",e.sqlstate
		 print "Error message:",e.msg
		 print "Error:",es = str(e)
		 print "Error:",s
	
errors.Error是Python StandardError的一个子类。

## 9.13.3 异常 errors.DataError

当数据有问题时，抛出这个异常。例如设置不能为NULL的列为NULL，超出列的取值范围，除以零，列数不匹配值数等等。
	
errors.DataError是errors.DatabaseError的一个子类。

## 9.13.4 异常 errors.DatabaseError

该异常是不匹配其他异常的任何MySQL错误的默认异常。
	
errors.DatabaseError是errors.Error的一个子类。

## 9.13.5 异常 errors.IntegrityError

当数据关系的完整性收到影响时，抛出该异常。例如，插入重复的键或者一个外键约束将失败。

下面的例子显示重复的键错误被以IntegrityError异常被抛出：

	 cursor.execute("CREATE TABLE t1 (id int,PRIMARY KEY (id))")
 	 try:
		 cursor.execute("INSERT INTO t1 (id) VALUES (1)")
		 cursor.execute("INSERT INTO t1 (id) VALUES (1)")
	 except mysql.connector.IntegrityError as err:
		 print("Error:{}".format(err))

errors.IntegrityError是DatabaseError的一个子类。

## 9.13.6 异常 errors.InterfaceError

源自Connector/Python本身的错误抛出该异常。与MySQL服务器无关。
	
errors.InterfaceError是errors.Error的一个子类。

## 9.13.7 异常 errors.InternalError

当MySQL服务器遇到一个内部错误时抛出该异常。例如，当发生死锁时。

errors.InternalError是errors.DatabaseError的一个子类。

## 9.13.8 异常 errors.NotSupportedError

当使用的一些特性在该MySQL版本下不支持返回错误时，抛出该异常。当使用函数或者语句不被储存例程支持时，也抛出该异常。
	
errors.NotSupportedError是errors.DatabaseError的一个子类。

## 9.13.9 异常 errors.OperationalError

与MySQL的操作相关的错误抛出该异常。例如，太多的连接，一个主机名不能被解析，失败的握手，服务器正被关闭，通信错误。
	
errors.OperationalError是errors.DatabaseError的一个子类。

## 9.13.10 异常 errors.PoolError

连接池错误抛出该异常。

errors.PoolError是errors.Error的一个子类。

## 9.13.11 异常 errors.ProgrammingError

编程错误抛出该异常，例如，当在你的SQL中有一个语法错误或者一个表没发现时。

下面的例子显示怎么处理语法错误：

	 try:
		 cursor.execute("CREATE DESK t1 (id int,PRIMARY KEY (id))")
	 except mysql.connector.ProgrammingError as err:
	 if err.errno == errorcode.ER_SYNTAX_ERROR:
		 print("Check your syntax!")
 	 else:
		 print("Error:{}".format(err))
	
errors.ProgrammingError是errors.DatabaseError的一个子类。
	
## 9.13.12 异常 errors.Warning

该异常用于报告重要的警告，然而，Connector/Python不使用它。它是为了兼容Python Database Specification v2.0(PEP-249)。

当你的查询产生警告时，考虑使用更严格的[Server SQL Modes](http://dev.mysql.com/doc/refman/5.6/en/sql-mode.html)或者[raise_on_warning](../07-ConnectorPython Connection Arguments/00-chapter7.md)连接参数来使得Connector/Python抛出错误。
	
errors.Warning是Python StandardError的一个子类。

## 9.13.13 函数 errors.custom_error_exception(error=None,exception=None)

该方法定义了为MySQL服务器错误自定义异常并返回当前的自定义。

如果error是一个MySQL服务器的错误号，你必须也需要传递exception类。error参数可以是一个字典，在这种情况下键是服务器错误号，值是被抛出的异常的类。
	
用空字典重置自定义：
	
	 import mysql.connectorfrom mysql.connector import errorcode

	 #服务器1028错误抛出一个DatabaseError
	 mysql.connector.custom_error_exception(1028,mysql.connector.DatabaseError)
	
	 #或者使用字典
	 mysql.connector.custom_error_excepiton({1028:mysql.connector.DatabaseError,1029:mysql.connector.OperationalError,})

	 #传递一个空字典，重置自定义
	 mysql.connector.custom_error_exception({})