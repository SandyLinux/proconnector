9.2 类connection.MySQLConnection
===================================

MySQLConnection类用于打开和管理一个MySQL服务器连接。它也用于发送命令和SQL语句及读取结果。

9.2.1 构造器 connection.MySQLConnection(**kwargs)

MySQLConnection构造器初始化属性，当至少一个参数传递时，它尝试去连接MySQL服务器。

完整的参数列表，查看[第7章 Connector/Python 连接参数](../07-ConnectorPython Connection Arguments/00-chapter7.md)。

9.2.2 属性 MySQLConnection.close()

语法：

	 cnx.close()

close()是disconnect()的同义词。查看[第9.2.20章](./02-chapter9.md)。

对于从连接池里获得的连接，close()实际上不关闭它，但是返回它到池中，使其可用于后续的连接请求。查看[第8.1章 Connector/Python 连接池](../08-ConnectorPython Other Topics/01-chapter8.md)。

9.2.3 方法 MySQLConnection.commit()

该方法发送一段COMMIT语句给MySQL服务器,提交当前的transaction。因为默认Connector/Python不会自动提交，因此在每个transaction后调用该方法是非常重要的。

	 >>>cursor.execute("INSERT INTO employees (first_name) VALUES (%s)",('Jane'))
	 >>>cnx.commit()

9.2.4 方法 MySQLConnection.config(**kwargs)

语法：

	 cnx.config(**kwargs)
	
配置一个MySQLConnection实例，在它被实例化之后。完整的可用参数列表，见[第7章 Connector/Python 连接参数](../07-ConnectorPython Connection Arguments/00-chapter7.md)。
	
参数：

- kwargs : 连接参数

你可以使用config()方法改变用户名，然后调用reconnect():

	 cnx = mysql.connector.connect(user='joe',database='test')
	 cnx.config(user='jane')
	 cnx.reconnect()

对于连接从连接池获得的连接，config()会抛出一个异常。

9.2.5 方法 MySQLConnection.connect()

语法：

	 MySQLConnection.connect(**kwargs)

该方法设置一个连接，建立与MySQL服务器的会话。如果没有给定参数，那么它将使用已经配置的或者默认值。

参数：

	 * kwargs 连接参数

例子：

	 cnx = MySQLConnection(user='joe',database='test')

对于从连接池获得的连接，连接对象的类是PooledMySQLConnection。一个pooled连接与一个unpooled连接不同，在[第8.1章 Connector/Python 连接池](../08-ConnectorPython Other Topics/01-chapter8.md)有描述。


9.2.6 方法 MySQLConnection.cursor(buffered=None,raw=None,cursor_class=None)

该方法返回一个MySQLCursor()对象，或者它的一个子类中，这取决于传递的参数。
	
当buffered是True时，操作执行后，该cursor抓取所有的行。当查询返回一个小的结果集时，这是很有用的。
	
设置raw，当抓取到行后，不会转换MySQL 数据类型为Python类型。Raw常用于你想要获得更好的性能或者你想要自己转换的时候。

cursor_class参数用于传递一个类用于实例化一个新cursor。它必须是cursor.CursorBase的一个子类。

返回对象依赖于buffered和raw参数的组合。

- 如果buffered和raw都没有:cursor.MySQLCursor
- 如果有buffered而没有raw:cursor.MySQLCursorBuffered
- 如果有buffered和raw都有:cursor.MySQLCursorBufferedRaw
- 如果没有buffered而有raw:cursor.MySQLCursorRaw

返回一个CursorBase实例。

9.2.7 方法 MySQLConnection.cmd_change_user(username='',password='',database='',charset=33)

使用uername和password改变用户，这也导致指定的database变成默认（当前的）database。它也可通过charset参数改变字符集。

返回一个字典包含ok包信息。

9.2.8 方法 MySQLConnection.cmd_debug()

指示服务器将调试信息写到错误日志。连接的用户必须要有SUPER权限。

返回一个字典包含ok包信息。

9.2.9 方法MySQLConnection.cmd_init_db(database)

该方法指定database为默认（当前）database。在随后的查询中，这个数据库是默认的表引用，包括没有明确的数据库限定符的。
	
返回一个字典包含OK包信息。

9.2.10 方法 MySQLConnection.cmd_ping()

检查连接的服务器是否正在工作。
	
该方法不能直接使用，用ping()或者is_connected()代替。

返回一个字典包含ok包信息。

9.2.11 方法 MySQLConnection.cmd_process_info()

该方法抛出NotSupportedError异常。使用SHOW PROCESSLIST语句或者查询数据INFORMATION_SCHEMA中的表代替。

9.2.12 方法 MySQLConnection.cmd_process_kill(mysql_pid)

告诉服务器杀死mysql_pid指定的线程。虽然还可用，但是使用KILL SQL语句更好。

返回一个包含OK信息包的字典。

	 >>>cnx.cmd_proccess_kill(123)
	 >>>cnx.cmd_query('KILL 123')

9.2.13 方法 MySQLConnection.cmd_query(statement)

该方法发送给定的statement到MySQL服务器并且返回一个结果。发送多条statement，使用cmd_query_iter()方法代替。
	
返回的字典包含的信息依赖哪种查询被执行。如果查询时SELECT语句，结果信息包含columns。其他语句返回一个字典包含OK或者EOF包信息。
	
从MySQL服务器接收到的错误会被作为异常抛出。当发现多个结果时，抛出一个InterfaceError异常。
	
返回一个字典。

9.2.14 方法 MySQLConnection.cmd_query_iter(statement)

类似于cmd_query()方法，但该方法返回一个生成器对象来遍历结果。当发送多个statement时，使用cmd_query_iter()，并且使用分号分隔。
	
下面这个例子说明了，发送多个statement后怎么遍历结果：
	
	 statement = 'SELECT 1;INSERT INTO t1 VALUES ();SELECT 2'
	 for result in cnx.cmd_query_iter(statement):
	 if 'columns' in result:
	  columns = result['columns']
	  rows = cnx.get_rows()
	 else:
	  #do something
	
返回一个生成器对象。

9.2.15 方法 MySQLConnection.cmd_quit()

该方法发送一个QUIT命令给MySQL服务器，关闭当前连接。因为从MySQL没有响应，因此将返回被发送的包。

9.2.16 方法 MySQLConnection.cmd_refresh(options)

该方法刷新表和缓存，或者重置replication服务器信息。连接的用户必须要有RELOAD权限。
	
options参数应该是一个位掩码值，使用constants.RefreshOption类的常量构建的。
	
options列表，见[第9.12章 类 constants.RefreshOption](../12-chapter9.md)

例子：

	 >>>from mysql.connector import RefreshOption
	 >>>refresh = RefreshOption.LOG | RefreshOption.THREADS
	 >>>cnx.cmd_refresh(refresh)

9.2.17 方法 MySQLConnection.cmd_reset_connection()

语法：

	 cnx.cmd_reset_connection()
	
通过发送一个COM_RESET_CONNECTION命令给服务器重置连接来清除会话状态。
	
该方法允许不重新认证来清除会话状态。对于早于MySQL5.7.3版本（COM_RESET_CONNECTION刚推出。），使用reset_session()方法代替。那个方法需要重新认证来重置会话状态，这花费更多资源。
	
该方法在Connector/Python 1.2.1中添加。

9.2.18 方法 MySQLConnection.cmd_shutdown()

该方法告诉数据库服务器关机。这要求连接的用户必须要有SHUTDOWN权限。

返回一个包含OK包信息的字典。

9.2.19 方法 MySQLConnection.cmd_statistics()

返回一个字典包含的信息是关于MySQL服务器的uptime,threads,questions,reloads,open tables。
	
9.2.20 方法 MySQLConnection.disconnect()

该方法尝试发送QUIT命令和关闭socket。它不抛出异常。MySQLConnection.close()是它的一个同义方法并且使用得更多。

9.2.21 方法 MySQLConnection.get_row()

该方法检索查询结果的下一行，返回一个元组。

通过get_row返回的元组的组成：

- 以包含比特对象的元组表示的行，或者当没有行的时候的None。
- 以包含status_flag,warning_count或者当返回的行不是最后一行时的None的字典表示的EOF包信息。

get_row()方法用于MySQLCursor抓取行。

9.2.22 方法 MySQLConnection.get_rows(count=None)

该方法取回一个查询结果集合的所有的或者剩余的行，返回一个元组包含行作为序列和EOF包信息。count参数用于获得给定数量的行。如果cout没有指定或者为None，将取回所有的行。
	
被get_rows()返回的元组的组成：

- 一个元组列表，元组中包含了以字节对象表示的行数据，或者当没有行的时候的空列表。
- 以包含status_flag和warning_count的字典表示的EOF包信息。

当所有的行已被取回后，抛出InterfaceError。

MySQLCursor使用get_rows()方法抓取行。

返回一个元组。

9.2.23 方法 MySQLConnection.get_server_info()

该方法逐字返回MySQL服务器信息作为字符串，例如'5.6.11-log',或者当没有连接时的None。

9.2.24 方法 MySQLConnection.get_server_version()

该方法返回以元组的形式返回MySQL的版本，或者当没有连接时为None。

9.2.25 方法 MySQLConnection.is_connected()

报告MySQL服务器连接是否可用。

该方法使用ping()方法检测MySQL连接是否可用，但是不像ping(),is_connected()返回True或者False。
	
9.2.26 方法 MySQLConnection.isset_client_flag(flag)

如果client flag设置了返回为True，否则为False。

9.2.27 方法 MySQLConnection.ping(attempts=1,delay=0)

检测MySQL服务器的连接是否仍然可用。

当reconnect设为True，使用reconnect()一次或者更多次attempts去重连MySQL服务器。如果你想要每次重试的时候等待段时间，可使用delay参数。

9.2.28 方法 MySQLConnection.reconnect(attempts=1,delay=0)

尝试重连MySQL服务器。

attempts参数指定重连的次数，delay参数指定每次重连等待的秒数。

当你期望的MySQL服务器停机维护或者网络临时不可用时，你可以设置attempts更高或者delay更长。

9.2.29 方法 MySQLConnection.reset_session()

语法：

	 cnx.reset_session(user_variables=None,session_variables=None)

通过重新认证来重置连接清除会话状态。如果给定user_variables，它是用户变量名称和值的字典，如果给定session_variables，它是系统变量名称和值的字典。该法设置每个变量为为给定的值。

例如：

	 user_variables = {'var1':'1','var2':'10'}
	 session_variables = {'wait_timeout':10000,'sql_mode':'TRADITIONAL'}
	 self.cnx.reset_session(user_variables,session_variables)

对于MySQL 5.7.3或者以后的版本，使用cmd_reset_connection()方法代替。

该方法在Connector/Python 1.2.1中添加。

9.2.30 方法 MySQLConnection.rollback()

该方法发送一个ROLLBACK语句给MySQL服务器，从当前事务中撤消所有数据的改变。默认的，Connector/Python不会自动提交，因此当使用的是事务性储存引擎如InnoDB时，是可能取消事务的。

	 >>>cursor.execute("INSERT INTO employees (first_name) VALUES (%s)",('Jane'))
	 >>>cnx.rollback()

提交修改，请查看commit()方法。

9.2.31 方法 MySQLConnection.set_charset_collation(charset=None,collation=None)

该方法设置charset集和collation用于当下连接。charset参数既可以是字符集的名称或者constants.CharacterSet中定义的等效数字。

当collation为None时，使用字符集的默认collation。

下面的例子，我们设置字符集为latin1和collation为latin_swedish_ci(默认的collation为latin1):

	 >>>cnx = mysql.connector.connect(user='scott')
	 >>>cnx.set_charset_collation('latin1')

指定collation如下：

	 >>>cnx = mysql.connector.connect(user='scott')
	 >>>cnx.set_charset_collation('latin1','latin1_general_ci')

9.2.32 方法 MySQLConnection.set_client_flags(flags)

该方法设置客户端标志来在连接MySQL服务器的时候使用,返回以整数的形式返回一个新值。flags参数既可以是一个整数或者合法客户端标志值的序列（查看[第9.8章，类 constants.ClientFlag(./08.chapter9.md)。）

如果flags是一个序列，序列中的每个值当它的值为正时设置标志，或者当它的值为负的时候取消设置。例如，取消设置LONG_FLAG 和设置FOUND_ROWS标志：

	 >>>from mysql.connector.constants import ClientFlag
	 >>>cnx.set_client_flags([ClientFlag.FOUND_ROWS,-ClientFlag.LONG_FLAG])
	 >>>cnx.reconnect()

注意:

	 客户端标志只能在连接服务器的时候设置和使用，因此作出改变后需要reconnect。

9.2.33 方法 MySQLConnection.start_transaction()

该方法开启一个事务。它接受参数指明是否使用consistent snapshot,使用哪种transaction isolation level，和transaction access mode:

	 cnx.start_transaction(consistent_snapshot=bool,isolation_level=level,readonly=access_mode)

默认的consistent_snapshot值是False。如果值为True，Connector/Python发送WITH CONSISTENT SNAPSHOT语句。MySQL对isolation level忽视它，该选项不适用。

默认的isolation_level的值为None，允许的值为'READ UNCOMMITTED','READ COMMITTED','REPEATABLE READ','SERIALIZABLE'。如果isolation_level的值为None，则没有隔离级别被发送，因此应用默认的级别。

readonly参数为True时，以READ ONLY模式开始事务，或者为False时，以READ WRITE模式开始事务。如果readonly省略，使用服务器的默认访问模式。详细的事务访问模式，查看START TRANSACTION,COMMIT,和ROLLBACK语法中START TRANSTACTION语句的描述。如果服务器的版本比5.6.5旧，不支持设置访问模式并且Connector/Python 抛出ValueError。

如果调用正在进行中的事务，调用start_transaction()会抛出一个ProgrammingError异常。这不同于在事务正在进行时执行START TRANSACTION SQL语句。该语句隐式提交当前事务。

决定事务对于连接是否处于活动状态，使用in_transaction属性。

start_transaction()在MySQL Connector/Python 1.1.0中添加。readonly参数在MySQL Connector/Python 1.1.5中添加

9.2.34 属性 MySQLConnection.autocommit

该属性可以被赋值True或False来开启或关闭MySQL自动提交的特征。该属性能被调用来获取当前自动提交的设置。

注意，当通过Connector/Python连接时，默认自动提交是关闭的。可使用autocommit 连接参数来开启。

当自动提交关闭以及使用事务存储引擎为InnoDB或者NDBCluster时，你必须提交事务。

	 >>>cnx.autocommit
	 False
	 >>>cnx.autocommit = True
	 >>>cnx.autocommit
	 True

9.2.35 属性 MySQLConnection.charset_name

该属性返回一个字符串说明连接使用哪种字符集，不管它是否连接。

9.2.36 属性 MySQLConnection.collation_name

该属性返回一个字符串说明连接使用哪种collation，不管它是否连接。

9.2.37 属性 MySQLConnection.connection_id

该属性为当前连接返回一个整数连接ID（进程ID或者会话ID）或者当没有连接时返回None。

9.2.38 属性 MySQLConnection.database

该属性通过USE语句设置当前的（默认的）数据库。它也可以用来获取当前数据库的名称。\

	 >>> cnx.database = 'test'
	 >>> cnx.database = 'mysql'
	 >>> cnx.database
	 u'mysql'

返回一个字符串。

9.2.39 属性 MySQLConnection.get_warnings

该属性可以被赋值为True和False来开启或者关闭是否警告应该被自动抓取。默认值是False。该属性能被调用用来获取当前的warnings设置。

当debugging 查询的时候，自动抓取警告是有用的。Cursors通过MySQLCursor.fetchwarnings()使警告可用。

	 >>>cnx.get_warnings = True
	 >>>cursor.execute('SELECT "a"+1')
	 >>>cursor.fetchall()
	 [(1.0,)]
	 >>>cursor.fetchwarnings()
	 [(u'Warning',1292,u"Truncated incorrect DOUBLE value:'a'")]
	
返回True或者False

9.2.40 属性 MySQLConnection.in_transaction

该属性返回True或者False来指明对于该连接一个事务是否处于活动状态。不管你是使用start_transaction() API调用或直接执行一个SQL语句如"START TRANSACTION 或者BEGIN"，它的值都为True。

 	 >>>cnx.start_transaction()
	 >>>cnx.in_transaction
	 True
	 >>>cnx.commit()
	 >>>cnx.in_transaction
	 False

in_transaction是在MySQL Connector/Python 1.1.0中加入。

9.2.41 属性 MySQLConnection.raise_on_warnings

该属性可以被赋值True或者False来开启或者关闭警告是否应该抛出异常。默认为False。该属性被调用来获取当前的异常设置。

设置raise_on_warnings也会设置get_warnings，因为警告需要被抓取，因此他们才能以异常被抛出。

注意，如果你想MySQL服务器直接报告警告为错误，你可能想要设置SQL模式（见[第9.2.44章 属性MySQLConnection.sql_mode](./02-chapter9.md)）。使用事务引擎也很好，这样当抓取到异常时，事务就能被回滚。

在任何异常被抛出前，结果集合需要被完全抓取。下面的例子是显示一条查询的执行产生一个警告：

	 >>>cnx.raise_on_warnings = True
	 >>>cursor.execute('SELECT "a"+1')
	 >>>cursor.fetchall()
	 ..
	 mysql.connector.error.DataError:1292:Truncated incorrect DOUBLE value :'a'

返回True或False

9.2.42 属性 MySQLConnection.server_host

该只读属性返回用于连接MySQL服务器的主机名称或者IP地址。

返回一个字符串。

9.2.43 属性 MySQLConnection.server_port

该只读属性返回用于连接MySQL服务器的TCP/IP端口。
	
返回一个整数。

9.2.44 属性 MySQLConnection.sql_mode
	
该属性用于为当前连接获取和设置SQL模式。这个值应是一个通过逗号隔开的不同模式的列表，或者模式的一个序列，最好使用constants.SQLMode类。

不设置所有模式，可以传一个空字符或者一个空序列。

	 >>>cnx.sql_mode = 'TRADITIONAL,NO_ENGINE_SUBSTITUTION'
	 >>>cnx.sql_mode.split(',')
	 [u'STRICT_TRANS_TABLES',u'STRICT_ALL_TABLES',u'NO_ZERO_IN_DATE',u'NO_ZERO_DATE',u'ERROR_FOR_DIVISION_BY_ZERO',u'TRADITIONAL',u'NO_AUTO_CREATE_USER',u'NO_ENGINE_SUBSTITUTION']

	 >>>from mysql.connector.constants import SQLMode
	 >>>cnx.sql_mode = [SQLMode.NO_ZERO_DATE,SQLMode.REAL_AS_FLOAT]
	 >>>cnx.sql_mode
	 u'REAL_AS_FLOAT,NO_ZERO_DATE'

返回一个字符串。

9.2.45 属性 MySQLConnection.time_zone

该属性用于对当前连接设置或者获取时区会话变量。

	 >>>cnx.time_zone = '+00:00'
	 >>>cur.execute('SELECT NOW()');cur.fetchone()
	 （datetime.datetime(2012,6,15,11,24,36),）
	 >>>cnx.time_zone = '-09:00'
	 >>>cur.execute('SELECT NOW()');cur.fetchone()
	 （datetime.datetime(2012,6,15,2,24,44),）
	 >>>cnx.time_zone
	 u'-09:00'

返回一个字符串。

9.2.46 属性 MySQLConnection.unix_socket

该只读属性返回用于连接该MySQL服务器的Unix socket文件。
	
返回一个字符串。

9.2.47 属性 MySQLConnection.user

该只读属性返回用于连接MySQL服务器的用户名。

返回一个字符串。