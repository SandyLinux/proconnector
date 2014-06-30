9.7 类 cursor.MySQLCursorPrepared
===================================

一种方法是执行一个prepared SQL语句是使用PREPARE和EXECUTE语句。然而，对于反复用不同的数据执行相同语句的不同执行，使用二进制客户端/服务器协议来发送和接收数据更有效。了解更多该协议的信息，查看[C API Prepared Statements](http://dev.mysql.com/doc/refman/5.6/en/c-api-prepared-statements.html).

第二种方法是创建一个cursor，在Connector/Python中开启用二进制协议prepared语句的执行。在两种情况下，连接对象的cursor()方法返回一个MySQLCursorPrepared对象，它继承至cursor.MySQLCursor。

- 简单的语法是在cursor()方法使用一个prepared=True参数，该语法在Connector/Python 1.1.2中可用。

 	 import mysql.connector
	 cnx = mysql.connector.connect(database='employees')
	 cursor = cnx.cursor(prepared=True)

- 另外，创建一个MySQLCursorPrepared类的实例使用cursor()方法的cursor_class参数。MySQLCursorPrepared在Connector/Python 1.1.0中可用。

	 import mysql.connector	
	 from mysql.connector.cursor import MySQLCursorPrepared
		
	 cnx = mysql.connector.connect(database='employees')
	 cursor = cnx.cursor(cursor_class=MySQLCursorPrepared)

一个MySQLCursorPrepared类实例的工作原理是这样的：

- 首先你传递一个语句给cursor的execute()方法，它预备这条语句。对于后续execute()的调用，如果statement相同，则准备阶段跳过。
- execute()方法带可选的第二个参数包含一个与statement中参数标记相关联的数据值的列表，他们必须是每个参数标记一个值。

例如：

	 cursor = cnx.cursor(prepared=True)
	 stmt = "SELECT fullname FROM employees WHERE id = %s" # (1)
	 cursor.execute(stmt,(5,))                             # (2)
	 # ... fetch data ...
	 cursor.execute(stmt,(10,))                            # (3)
	 # ... fetch data ...

(1) statement中的%s是一个参数标记，不要放引号吧参数标记括起来。
(2) 对于第一次调用execute()方法，cursor准备该statement。如果数据是在同一个调用中给出，它也执行该statement，你应该抓取数据。
(3) 对于后续的execute()调用，传递相同的SQL statement，cursor将跳过准备阶段。

被MySQLCursorPrepared执行的prepared statement可以使用format(%s)或者qmark(?)的参数化风格。它与被MySQLCursor执行的nonprepared statement的不同的地方是，哪个能使用format或者pyformat参数化风格。

同时使用多条prepered statements，要从MySQLCursorPrepared类实例化multiple cursors。