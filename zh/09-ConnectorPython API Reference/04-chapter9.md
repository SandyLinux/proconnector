9.4 类 pooling.PooledMySQLConnection

该类用于MySQLConnectionPool返回一个pooled连接实例。It is also the class used for connections obtained with calls to the connect() method that name a connection pool（查看[第8.1章 Connector/Python 连接池](../08-ConnectorPython Other Topics/01-chapter8.md)）

PooledMySQLConnection pooled 连接对象是类似于MySQLConnection unpooled 连接对象，有以下的不同点：

- 释放一个从连接池获得的pooled连接，调用它的close()方法，如任何的unpooled连接一样。然而，对于一个pooled连接，close()其实不关闭连接而是返回它到池，使得它对后续的连接请求可用。
-  一个pooled连接不能被使用config()重新配置。改变连接必须通过池对象本身来完成，稍后会有描述。
- 一个pooled连接有一个pool_name属性返回池名。

## 9.4.1 构造器 pooling.PooledMySQLConnection

语法：
	
	 PooledMySQLConnection(cnxpool,cnx)
	
该构造器带一个连接池和连接参数，并返回一个pooled的连接。它被MySQLConnectionPool类使用。
	
参数： 

- cnxpool:一个MySQLConnectionPool实例。
- cnx:一个MySQLConnection实例。
	
例子：

	 pcnx = mysql.connector.pooling.PooledMySQLConnection(cnxpool,cnx)

## 9.4.2 方法 PooledMySQLConnection.close()

语法：

	 cnx.close()
	
返回一个pooled连接到它的连接池。
	
对于一个pooled的连接，close()实际上并不会关闭连接，而是返回它到连接池，使得对后续的连接请求可用。
	
如果pool配置参数改变了，一个返回的连接被关闭并且使用新的配置重新打开，before being returned from the pool again in response to a connection request.

## 9.4.3 方法 PooledMySQLConnection.config()

对于pooled连接，config()方法抛出一个PoolError异常。为pooled连接配置应该通过使用pool对象完成。

## 9.4.4 属性 PooledMySQLConnection.pool_name

语法：

	 cnx.pool_name
	
该属性返回该连接所属连接池的名称。
	
例子：
	
	 cnx = cnxpool.get_connection()name = cnx.pool_name