﻿9.6 类 cursor.MySQLCursorBuffered
====================================

这个类从cursor.MySQLCursor继承而来，如果有需要，在operation被执行后自动接收行。

MySQLCursorBuffered在有两个查询的情况下很有用，小结果集，需要彼此组合或者计算。你可以在用连接的cursor()方法的时候使用buffered参数，或者使用buffered连接参数来默认为所有创建的cursors生成connection buffering。

```python
	
	 import mysql.connectorcnx = mysql.connector.connect()

	 #仅仅这个特定的cursor将缓存结果	
	 cnx.cursor(buffered=True)
		
	 #所有的cursors将默认被缓存	
	 cnx2 = mysql.connector.connect(buffered=True)
```
	
一个典型的使用案例，请见[第6.1章 教程：使用一个Buffering Cursor提取出员工的薪水](../06-ConnectorPython Tutorials/01-chapter6.md))