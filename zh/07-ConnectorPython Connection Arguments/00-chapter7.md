第7章 Connector/Python 连接参数
==================================
可以用mysql.connector.connect()函数或者mysql.connector.MySQLConnection()类建立与MySQL服务器的连接。

	 cnx = mysql.connector.connect(user='joe',database='test')
	 cnx = MySQLConnection(user='joe',database='test')	

下表描述的参数用于启动连接。带星号的参数表明同义的参数名，只是为兼容其他Python的MySQL驱动。Oracle建议不要使用这些替代名。

	
表7.1 Connector/Python的连接参数

	 +------------------+------------------+--------------------------------------------------+
	 |Argument Name     |  Default         |   Description                                    |
	 +------------------+------------------+--------------------------------------------------+
	 |user(username*)   |                  |  用于MySQL服务进行身份认证的用户名。             |
	 +------------------+------------------+--------------------------------------------------+
	 |password(passwd*) |                  |  用于MySQL服务进行身份认证用户名的密码。         |
	 +------------------+------------------+--------------------------------------------------+
	 |database(db*)     |                  |  当连接MySQL服务器时，使用的数据库名称。         |
	 +------------------+------------------+--------------------------------------------------+
	 |host              |  127.0.0.1       |  MySQL服务器的ip地址或者主机名称。               |
	 +------------------+------------------+--------------------------------------------------+
	 |port              |  3306            |  MySQL服务器的TCP/IP端口号。必须是整数。         |
	 +------------------+------------------+--------------------------------------------------+
	 |unix_socket       |                  |  Unix套接字文件的位置。                          |
	 +------------------+------------------+--------------------------------------------------+
	 |use_unicode       |  True            |  是否使用Unicode。                               |
	 +------------------+------------------+--------------------------------------------------+
	 |charset           |  utf8            |  使用哪种MySQL字符集                             |
	 +------------------+------------------+--------------------------------------------------+
	 |collaction        |  utf8_general_ci |  使用哪种MySQL collation                         |
	 +------------------+------------------+--------------------------------------------------+
	 |autocommit        |  False           |  是否autocommit transactions。                   |
	 +------------------+------------------+--------------------------------------------------+
	 |time_zone         |                  |  在连接时，设置time_zone会话变量。               |
	 +------------------+------------------+--------------------------------------------------+
	 |sql_mode          |                  |  在连接时，设置sql_mode会话变量。                |
	 +------------------+------------------+--------------------------------------------------+
	 |get_warnings      |  False           |  是否抓取警告                                    |
	 +------------------+------------------+--------------------------------------------------+
	 |raise_on_warnings |  False           |  在发生警告时，是否抛出异常。                    |
	 +------------------+------------------+--------------------------------------------------+
	 |connection_timeout|                  |  TCP和Unix socket连接的超时时间                  |
	 |(connect_timeout*)|                  |                                                  |
	 +------------------+------------------+--------------------------------------------------+
	 |client_flags      |                  |  MySQL客户端标志                                 |
	 +------------------+------------------+--------------------------------------------------+
	 |buffered          |  False           |  在执行查询后，cursor对象是否立即抓取结果。      |
	 +------------------+------------------+--------------------------------------------------+
	 |raw               |  False           |  是否原样返回MySQL的结果，而不转换为Python类型。 |
	 +------------------+------------------+--------------------------------------------------+
	 |ssl_ca            |                  |  包含SSL certificate authority的的文件。         |
	 +------------------+------------------+--------------------------------------------------+
	 |ssl_cert          |                  |  包含SSL certificate file的文件。                |
	 +------------------+------------------+--------------------------------------------------+
	 |ssl_key           |                  |  包含SSL key的文件。                             |
	 +------------------+------------------+--------------------------------------------------+
	 |ssl_verify_cert   |  False           |  当设置为True时，检查由ssl_ca选项指定的证书文件。|
	 |                  |                  |  任何不匹配会导致一个ValueError的异常。          |
	 +------------------+------------------+--------------------------------------------------+
	 |force_ipv6        |  False           |  当设置为True时，当地址解析为IPv4和IPv6时，      |
	 |                  |                  |  使用IPv6。默认使用IPv4                          |
	 +------------------+------------------+--------------------------------------------------+
	 |dsn               |                  |  不支持（当使用时，抛出NotSupportedError异常）   |
	 +------------------+------------------+--------------------------------------------------+
	 |pool_name         |                  |  连接pool名称。1.1.1增加。                       |
	 +------------------+------------------+--------------------------------------------------+
	 |pool_size         |  5               |  连接pool大小。1.1.1增加。                       |
	 +------------------+------------------+--------------------------------------------------+
	 |pool_reset_session|  True            |  当连接返回pool时，是否重置会话变量。1.1.5增加。 |
	 +------------------+------------------+--------------------------------------------------+
	 |compress          |  False           |  是否使用compressed 客户端/服务器 协议。1.1.2增加|
 	 +------------------+------------------+--------------------------------------------------+
	 |converter_class   |                  |  使用converter类。1.1.2增加。                    |
 	 +------------------+------------------+--------------------------------------------------+
	 |fabric            |                  |  MySQL Fabric 连接参数。1.2.0增加。              |
	 +------------------+------------------+--------------------------------------------------+
	 |auth_plugin       |                  |  使用验证插件。1.2.1增加。                       |
	 +------------------+------------------+--------------------------------------------------+
	 |failover          |                  |  服务器failover序列。1.2.1增加。                 |
	 +------------------+------------------+--------------------------------------------------+

## MySQL 认证

MySQL认证是使用username和password参数。

注意：
	 MySQL Connector/Python不支持MySQL版本4.1以前的老的，不太安全的密码协议。
	 
当给定了database参数，当前的数据库会被设置为给定的值。以后要改变当前的数据库，执行USE SQL 语句或者设置MySQLConnection实例的database属性。

默认，MySQL Connector/Python会使用TCP/IP试图连接运行在本地主机的MYSQL服务器。host参数默认为IP地址127.0.0.1和port参数默认为3306。通过设置unix_socket来支持Unix sockets。在windows平台不支持named pipes。

## 认证插件

MySQL Connector/Python 1.2.1和以上支持MySQL 5.6中的认证插件。它包含mysql_clear_password和sha256_password,它们都需要一个SSL连接。sha256_password不能工作在一个非SSL连接上，因为Connector/Python不支持RSA加密。

connect()方法支持一个auth_plugin参数用于强制使用某一特定的插件。例如，如果服务器被配置为默认使用sha256_password，你想要连接一个帐号使用mysql_natvie_password认证，你可以使用SSL连接或者指定auth_plugin="mysql_native_password"。

## 字符编码

默认，来自MySQL的字符串被以Python Unicode文字返回。设置user_unicode为False，可以改变该行为。你可以通过客户端连接的charset参数来改变字符设置。要改变已连接到MySQL的字符设置，可设置MySQLConnection实例的charset属性。该技术优于直接使用SET NAMES SQL语句。相似于charset属性，你可以为当前的MySQL会话设置collation。

## 事务

autocommit值默认为False，因此事务不会自动提交。在你的应用中做插入，更新，删除操作后，调用MySQLConnection实例的commit()方法。对于数据的一致性和高吞吐量的写操作，当使用InnoDB或者其他事务性表的时候，最好保持autocommit配置选项关闭。

## 时区

可以使用time_zone参数为每个连接设置时区。这是非常有用的，例如，如果MySQL服务器设置为UTC，被MySQL返回的TIMESTAMP值会被转换为PST时区。

## SQL模式

MySQL支持所谓的SQL模式。它会改变服务器全局的或者每个连接的行为。例如，警告被以错误抛出，设置sql_mode为TRADITIONAL。了解更多信息，查看[Server SQL Mode](http://dev.mysql.com/doc/refman/5.6/en/sql-mode.html)。

## 故障排除和错误处理

当get_warnings设置为True时，会自动抓取查询所产生的警告。你也可以通过设置raise_on_warnings为True来立即抛出有一个异常。请考虑使用MySQL sql_mode设置来转变警告为错误。

使用connection_timeout参数来为一个连接设置一个超时值。

## 使用客户端标志来开启和关闭特征

MySQL使用client flags来开启和关闭特征。使用client_flags参数，你控制设置什么。要查看哪些标志可用，使用下列代码：

	 from mysql.connector.constants import ClientFlag
	 print '\n'.join(ClientFlag.get_full_info()
	 
如果client_flags没有指定（即，为0），MySQL v4.1和以后版本使用默认值。如果你指定一个大于0的整数，确保所有的标志都设置正确。一种更好的方法来设置和不设置个别标志是使用列表。例如，设置FOUND_ROWS,关闭默认的LONG_FLAG:

	 flags = [ClientFlag.FOUND_ROWS,-ClientFlag.LONG_FLAG]
	 mysql.connector.connect(client_flags=flags)
	 
## 缓存结果集

默认，MySQL Connector/Python不缓存或预取结果。这意味着执行一个查询后，你的程序抓取数据。这就避免了当查询返回大结果集的时候使用过多的内存。如果你知道结果集小到足以一次处理完。你可通过设置buffered为True来立即抓取结果。也可以为每个cursor设置它(查看[第9.2.6章 方法 MySQLConnection.cursor(buffered=None,raw=None,cursor_class=None)](../09-ConnectorPython API Reference/02-chapter9.md) )

## 类型转换

默认，结果集中的MySQL类型会被自动转换为Python对象。例如，一个DATETIME列值将变为datetime.datetime对象。设置raw参数为True，可以关闭转换。这么做你可以获得更好的性能或者自己执行不同类型的转换。

## 通过SSL连接

当你的[python安装支持SSL](https://docs.python.org/2/library/ssl.html)，即，当它兼容OpenSSL库时，可以使用SSL连接。当你提供ssl_ca,ssl_key,ssl_cert参数时，连接转换到SSL，并且client_flags选项自动包含ClientFlags.SSL值，你可以用它与compressed设置为True组合使用。

对于Connector/Python 1.2.1，可用仅用ssl_ca参数来建立SSL连接。ssl_key和ssl_cert参数是可选的。然而，当一个给定时，另一个也必须给定，否则抛出一个AttributeError。

	 # Note (Example is valid for Python v2 and v3)
	 from __future__ import print_function
	 
	 import sys
	 
	 #sys.path.insert(0, 'python{0}/'.format(sys.version_info[0]))
	 
	 import mysql.connector
	 from mysql.connector.constants import ClientFlag
	 config = {
	 'user': 'ssluser',
	 'password': 'asecret',
	 'host': '127.0.0.1',
	 'client_flags': [ClientFlag.SSL],
	 'ssl_ca': '/opt/mysql/ssl/ca-cert.pem',
	 'ssl_cert': '/opt/mysql/ssl/client-cert.pem',
	 'ssl_key': '/opt/mysql/ssl/client-key.pem',
	 }
	 cnx = mysql.connector.connect(**config)
	 cur = cnx.cursor(buffered=True)
	 cur.execute("SHOW STATUS LIKE 'Ssl_cipher'")
	 print(cur.fetchone())
	 cur.close()
	 cnx.close()
	 
## 连接池

无论出现pool_name或者是pool_size参数，Connector/Python创建一个新的连接池。如果pool_name参数没有给，connect()自动生成名称，该名称由给出的连接参数host,port,user,database以该顺序组成。如果pool_size参数没有给出，默认为5。

pool_reset_session允许控制当连接返回到连接池时，会话变量是否被重置。默认是重置他们。

连接池在Connector/Python 1.1.1中支持，查看[第8.1章 Connector/Python 连接池](../08-ConnectorPython Other Topics/01-chapter8.md)

## 协议压缩

布尔型compress参数指明是否使用压缩的client/server协议（默认为False）。它提供了一种更简便的替代方法是设置ClientFlag.COMPRESS标志。该参数在Connector/Python 1.1.2中可用。

## Converter 类

converter_class参数需要一个类，并且在配置连接的时候使用它。如果自定义的converter类不是conversion.MySQLConverterBase类的一个子类，则抛出一个AttributeError。该参数在Connector/Python 1.1.2 中可用。在1.1.2之前，仅在实例化一个新连接对象后，设置自定义converter类才可用。

## MySQL Fabric 支持

请求MySQL fabric连接，提供一个fabric参数来指定联系Fabric。
更多细节，查看[Request a Fabric Connection](http://dev.mysql.com/doc/mysql-utilities/1.4/en/connector-python-fabric-connect.html)。

## 服务器故障转移

Connector/Python 1.2.1，connect()方法接受一个failover参数，它提供信息用于在连接失败事件中服务器故障转移。该参数值是一个字典元组或者字典列表列表（最好是元组，因为它是不可变的）。每个字典包含在故障切换序列中给定的服务器的连接参数。允许的字典值是：user,password,host,port,unix_socket,database,pool_name,pool_size。

## 与其他连接接口的兼容

为了兼容其他MySQL接口，passwd,db和connect_timeout有效的，它们与相应的password,database,connection_timeout一样。后者优先。数据源名称语法或者dsn不能使用，如果指定，抛出NotSupportedError异常。