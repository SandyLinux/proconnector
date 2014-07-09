9.3 类 pooling.MySQLConnectionPool
=====================================

该类提供了连接池的实例化和管理。
	
## 9.3.1 构造器 pooling.MySQLConnectionPool

语法：

```python

	 MySQLConnectionPool(pool_name=None, pool_size=5, pool_reset_sessions=True,**kwargs)
```
	
该构造器实例化一个对象来管理一个连接池：
参数：

- pool_name: 连接池名称。
  如果该参数没给，Connector/Python会自动生成名称。由kwargs中给出的host,port,user和database连接参数以该顺序组成。对于多个池有相同的名称不会被认为是一个错误。一个应用应该用它们的pool_name属性来区分不同的池，应该为每个池创建一个不同的名称。	
- pool_size:池大小，如果没有给出，默认是5。
- pool_reset_session：当连接被返回到池的时候，是否重置会话变量。

  该参数在Connector/Python 1.1.5中添加。1.1.5以前，会话变量不会被重置。
- kwargs:可选额外连接参数，如[第7章 Connector/Python 连接参数](../07-ConnectorPython Connection Arguments/00-chapter7.md)中描述。

例子：

```python

	 dbconfig = {"database":"test","user":"joe",}
	 cnxpool = mysql.connector.pooling.MySQLConnectionPool(pool_name = "mypool",pool_size = 3,**dbconfig)
```

## 9.3.2 方法 MySQLConnectionPool.add_connection()

语法：
	 
```python

	 cnxpool.add_connection(cnx = None)
```

该方法添加一个新的或已存在的MySQLConnection到池，或者如果池满了抛出一个PoolError。

参数：
- cnx:MySQLConnection对象被添加到池。如果该参数没有给，则该池就新增一个连接并添加它。

例子:

```python

	 cnxpool.add_connection()    #添加一个新连接到池
	 cnxpool.add_connection(cnx) #添加存在的连接到池
```

## 9.3.3 方法 MySQLConnectionPool.get_connection()

语法：

```python

	 cnxpool.get_connection()
```

该返回从连接池返回一个连接，或如没有连接可用则抛出一个PoolError。

例子：

```python
	
	 cnx = cnxpool.get_connection()
```

## 9.3.4 方法 MySQLConnectionPool.set_config()

语法：

```python
	 
	 cnxpool.set_config(**kwargs)
```

该方法为pool中的连接设置配置参数。配置更改后，pool中的连接请求使用新参数。改变之前所获得的连接不受影响。但是，当他们被关闭（返回pool中）使用新参数重新打开，在被pool为后续连接请求返回之前。

参数：

- kwargs：连接参数。

例子：

```python

 	 dbconfig = {
	 "database":"performance_schema",
	 "user":    "admin",
	 "password":"secret",
	 }

	 cnxpool.set_config(**dbconfig)
```

## 9.3.5 属性 MySQLConnectionPool.pool_name

语法：

```python

	 cnxpool.pool_name
```
	
该属性返回连接池名。
	
例子：

```python

	 name = cnxpool.pool_name
```