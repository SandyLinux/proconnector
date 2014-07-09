9.8 类 constants.ClientFlag
==============================

这个类提供MySQL 客户端标志的定义的常量，能被用于在连接建立时配置会话。当导入mysql.connector时，ClientFlag类可用。

```python

 	 >>>import mysql.connector
	 >>>mysql.connector.ClientFlag.FOUND_ROWS
```

查看[第9.2.32章 方法 MySQLConnection.set_client_flags(flags)](./02-chapter9.md)和[连接参数](../07-ConnectorPython Connection Arguments/00-chapter7.md)client_flag。
	
ClientFlag类不能被实例化。