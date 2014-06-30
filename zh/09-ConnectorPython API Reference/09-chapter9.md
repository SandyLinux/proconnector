9.9 类 constants.FieldType
============================

该类提供所有支持的MySQL field 或者数据类型。当处理原始数据或者定义你自己的转换器的时候，他们是有用的。field type随着每个cursor被储存在每个列的描述中。

下面的例子显示怎样在结果集中为每个列打印数据类型的名称。

	 from __future__ import print_function
	 import mysql.connector
	 from mysql.connector import FieldType

	 cnx = mysql.connector.connect(user='scott',database='test')
	 cursor = cnx.cursor()

	 cursor.execute("SELECT DATE(NOW()) AS `c1`,TIME(NOW()) AS `c2`,"
				    "NOW() AS `c3`,'a string' AS `c4`,42 AS `c5`")

	 rows = cursor.fetchall()
	 for desc in cursor.description:
	  colname = desc[0]
	  coltype = desc[1]
	  print("Column {} has type {}".format(colname,FieldType.get_info(coltype)))
	 cursor.close()
	 cnx.close()

FieldType类不能被实例化。