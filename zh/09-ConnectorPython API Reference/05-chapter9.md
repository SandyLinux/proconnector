9.5 类 cursor.MySQLCursor
============================

MySQLCursor类用于实例化执行操作的对象，例如SQL查询。他们使用MySQLConnection对象与MySQL服务器进行交互。

这些相关的类是从cursor.MySQLCursor继承：

- cursor.MySQLCursorBuffered创建一个被缓存的cursor。查看第9.6章，"类 cursor.MySQLCursorBuffered"。
- cursor.MySQLCursorPrepared为准备执行的语句创建一个cursor。查看第9.7章，"类 cursor.MySQLCursorPrepared"。

## 9.5.1 构造器 cursor.MySQLCursor

该构造器带可选项connection初始化实例，该connection参数应该是MySQLConnection类的一个实例。

在多数情况下，`MySQLConnection.cursor()`方法用于实例化一个MySQLCursor对象。

## 9.5.2 方法 MySQLCursor.callproc(procname,args=())

该方法调用给定名称的储存过程。参数args序列必须包含每个参数一个条目，如常规预期的。结果是以输入序列修改过的副本返回。输入参数保持不变。输出和输入/输出参数可能被新值替代。

储存过程产生的结果集是自动抓取和以MySQLCursorBuffered实例储存。了解更多信息，查看stored_results()。

下面的例子展示怎样执行一个带两个参数的的储存过程，值相乘并返回乘积：

```python

	 #乘法储存过程的定义：
	 #CREATE PROCEDURE multiply(IN pFac1 INT,IN pFac2 INT,OUT pProd INT)
	 #BEGIN
	 # SET pProd := pFac1 * pFac2;
	 #END

	 >>>args = (5,5,0)
	 >>>cursor.callproc('multiply',args)
	 ('5','5','5L')
```

Connector/Python1.2.1和以上允许指定参数类型。要做到这点，就要指定一个参数为一个两元素的元组包含参数的值和类型。假设一个过程sp1()有这样的定义：

```sql

	 CREATE PROCEDURE sp1(IN pStr1 VARCHAR(20),IN pStr2 VARCHAR(20),OUT pConCat VARCHAR(100))
	 BEGIN
	  SET pConCat := CONCAT(pStr1,pStr2);
	 END
```

从Connection/Python中执行这个过程，指定OUT参数的类型，这样做：

```python

	 args = ('ham','spam',(0,'CHAR'))
	 cursor.callproc('sp1',args)
	 print(cursor.fetchone())
```

## 9.5.3 方法 MySQLCursor.close()

该方法关闭MySQL cursor，重置所有结果，并确保cursor对象没有引用的连接对象。

你每次使用完成cursor后用close()

## 9.5.4 方法 MySQLCursor.execute(operation,params=None,multi=False)

该方法准备给定的数据operation（查询或者命令）。元组中发现的参数或者字典params被绑定到operation中的变量。变量被以%s(format)或者%(name)s(pyformat)参数风格指定。
	
这个例子插入一个新雇员，然后选择他的数据：

```python

	 insert = ("INSERT INTO employees (emp_no,first_name,last_name,hire_date)"          
	 		   "VALUES (%s,%s,%s,%s)")
	 data = (2,'Jane','Doe',datetime.date(2012,3,23))cursor.execute(insert,data)
	 select = "SELECT * FROM employees WHERE emp_no = %(emp_no)s"
	 cursor.execute(select,{'emp_no':2})
```
	
注意:

	 数据值必要时需要将Python对象转换为MySQL明白的东西。在上个例子中，datetime.date()实例被转换为'2013-03-23'。

当multi被设置为True时，execute()可执行多个语句。它返回一个迭代器使得它能够为每个语句处理结果。注意，在这种情况下，使用参数不能很好的工作。在它自己上执行每个语句常是个好主意。
	
下面的例子在一个operation中选择和插入数据并显示结果：

```python

	 operation = 'SELECT 1;INSERT INTO t1 VALUES();SELECT 2'
	 for result in cursor.execute(operation):
	 if result.with_rows:
	 	print("Statement '{}'has following rows:".format(result.statement))
	 	print(result.fetchall())
	 else:
	 	print("Affected row(s) by query '{}' was {}".format(result.statement,result.rowcount))
```

如果连接被配置为抓取警告，被operation生成的警告通过MySQLCursor.fetchwarnings()方法可用。

当multi为True时，返回一个迭代器。

## 9.5.5 方法 MySQLCursor.executemany(operation,seq_parpams)

这个方法准备一个数据库operation（查询或者命令），然后对参数序列或者seq_params中的映射执行它。

executemany()通过迭代参数序列调用execute()方法。然而，插入数据可通过使用multipler-row 语法批量处理优化。

下面这个例子插入3条记录：

```python

	 data = [('Jane',date(2005,2,12)),('Joe',date(2006,5,23)),('John',date(2010,10,3)),]
	 stmt = "INSERT INTO employees(first_name,hire_date) VALUES(%s,%s)"
	 cursor.executemany(stmt,data)
```

在上面的例子中，INSERT语句发送给MySQL会是：

```sql

	 INSERT INTO employees(first_name,hire_date)	VALUES('Jane','2005-02-12'),('Joe','2006-05-23'),('John','2010-10-03')
```

不可以使用executemany()方法执行多条语句。这样做会抛出一个InternalError异常。

## 9.5.6 方法 MySQLCursor.fetchall()

该方法抓取所有或者剩余的查询结果集的行，以元组列表的形式返回。当没有行的时候，返回一个空列表。

下面的例子展示怎么获取第一个2行结果，然后获取剩余的行：

```python

	 >>>cursor.execute("SELECT * FROM employees ORDER BY emp_no")
	 >>>head_rows = cursor.fetchmany(size=2)
	 >>>remaining_rows = cursor.fetchall()
```
	
你必须在用相同连接执行新查询的之前抓取所有的行。

## 9.5.7 方法 MySQLCursor.fetchmany(size=1)

该方法抓取一个查询结果的下个集合，返回一个元组的列表。当没有更多行可用时，返回一个空列表。

返回的行数有size参数指定，默认为1。当没有更多的行可用时，返回的行可能会少于参数指定的值。

注意:
	
	 你必须在用相同连接执行新查询的之前抓取所有的行。

## 9.5.8 方法 MySQLCursor.fetchone()

该方法获取查询结果集的下行，返回单个序列，或者当没有更多行可用时返回None。返回的元组包含MySQL服务器返回数据转换为Python对象。

fetchone()方法被用于fetchall()和fetchmany()。也用于当使用MySQLCursor实例作为迭代器时。
	
下面的例子展示了怎么使用fetchone()来处理一个查询结果，首先使用一个while循环，然后使用迭代器：

```python
	
	 #使用一个while循环
	 cursor.execute("SELECT * FROM employees")
	 row = cursor.fetchone()
	 while row is not None:
	 	print(row)
	 	row = cursor.fetchone()
		
	 #使用cursor作为迭代器
		
	 cursor.execute("SELECT * FROM employees")
	 for row in cursor:
	 	print(row)
```

	你必须在用相同连接执行新查询的之前抓取所有的行。

## 9.5.9 方法 MySQLCursor.fetchwarnings()

该方法返回一个元组列表包含上一个执行语句生成的警告。使用连接的get_warnings属性来设置是否警告应被抓取。
下面的例子展示一个SELECT语句生成一个警告。

```python
	
	 >>> cnx.get_warnings = True
	 >>> cursor.execute('SELECT "a"+1')
	 >>> cursor.fetchall()
	 [(1.0,)]
	 >>> cursor.fetchwarnings()
	 [(u'Warning', 1292, u"Truncated incorrect DOUBLE value: 'a'")]
```

当发现警告时，有可能抛出错误。查看MySQLConnection.raise_on_warnings属性。

## 9.5.10 方法 MySQLCursor.stored_results()

该方法返回一个列表迭代器对象，用于处理一个用callproc()方法调用储存过程后产生的结果集。

下面的例子执行一个储存过程，产生两个结果集，然后用stored_results()获取他们：

```python
	
	 >>>cursor.callproc('sp1')()
	 >>>for result in cursor.stored_results():
	  print result.fetchall()
	 [(1,)][(2,)]
```

结果集一直可用，直到你执行另外的操作或者你调用另外的储存过程。

## 9.5.11 属性 MySQLCursor.column_names

这个只读属性以Unicode字符串序列的形式返回结果集的列名。

下面的例子展示怎么从包含数据和用column_names为键的元组中创建一个字典：

```python
	
	 cursor.execute("SELECT last_name,first_name,hire_date "             
	                "FROM employees WHERE emp_no = %s",(123,))
	 row = dict(zip(cursor.column_names,cursor.fetchone())
		
	 print("{last_name},{first_name}:{hire_date}".format(row))
```

## 9.5.12 属性 MySQLCursor.description

这个只读属性返回一个描述结果集中列的元组列表。每个元组包含值如下：

```python

	 (column_name,
	 type,
	 None,
	 None,
	 None,
	 None,
	 null_ok,
	 column_flags)
```

下面的例子展示怎样解释description元组：

```python

	 import mysql.connector
	 from mysql.connector import FieldType
	 ...
	 cursor.execute("SELECT emp_no, last_name, hire_date "
	 			    "FROM employees WHERE emp_no = %s", (123,))
		
	 for i in range(len(cursor.description)):
	 	print("Column {}:".format(i+1))
	 	desc = cursor.description[i]
	 	print("column_name = {}".format(desc[0]))
	 	print("type = {} ({})".format(desc[1], FieldType.get_info(desc[1])))
	 	print("null_ok = {}".format(desc[6]))
	 	print("column_flags = {}".format(desc[7]))
```

输出结果：

```python

	 Column 1:
	 column_name = emp_no
	 type = 3 (LONG)
	 null_ok = 0
	 column_flags = 20483
	 Column 2:
	 column_name = last_name
	 type = 253 (VAR_STRING)
	 null_ok = 0
	 column_flags = 4097
	 Column 3:
	 column_name = hire_date
	 type = 10 (DATE)
	 null_ok = 0
	 column_flags = 4225
```

column_flags值是constants.FieldFlag类的一个实例。查看怎么解释它，如下：

```python
	
	 >>>from mysql.connector import FieldFlag
	 >>>FieldFlag.desc
```

## 9.5.13 属性 MySQLCursor.lastrowid

该只读属性返回最新修改行的row ID。例如，如果你在一个表中插入一条记录，该表包含一个AUTO_INCREMENT列，lastrowid返回这个新行的AUTO_INCREMENT值。查看[第5.3章 使用Connector/Python插入数据](../05-ConnectorPython Coding Examples/03-chapter5.md)。

## 9.5.14 属性 MySQLCursor.statement

该只读属性以字符串的形式返回最后执行的语句。该字符串能包含多条语句，如果一个多条语句字符串被执行的话。

该statement属性对应调试和显示什么被发送到MySQL服务器了很有用。

## 9.5.15 属性 MySQLCursor.with_rows

当执行的操作的结果提供行，该只读属性返回True。

当有必要决定是否一条语句产生一个结果集和你需要抓取行时，with_rows属性很有用。下面的例子获取SELECT语句返回的行，但是仅仅报告UPDATE语句影响的行的值。

```python

	 import mysql.connector
	 cnx = mysql.connector.connect(user='scott', database='test')
	 cursor = cnx.cursor()
	 operation = 'SELECT 1; UPDATE t1 SET c1 = 2; SELECT 2'
	 for result in cursor.execute(operation, multi=True):
	  if result.with_rows:
	   result.fetchall()
	  else:
	   print("Updated row(s): {}".format(result.rowcount))
```