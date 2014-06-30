第9章 Connector/Python API 参考
==================================

本章包含Connector/Python的公共API参考。例子应该考虑工作在Python 2.7，和Python3.0及以上。它们也可以在旧版本上工作（例如Python 2.4），除非它们使用较新的Python版本中引入的功能。例如，异常处理使用as关键字是在Python 2.6中引入，不能在Python 2.4中正常工作。

以下概述显示了mysql.connector包和它的模块。目前，仅记载了对最终用户最有用的模块、类和方法。

	 mysql.connector
	 errorcode
	 errors
	 connection
	 constants
	 conversion
	 cursor
	 dbapi
	 locales
	  eng
	   client_error
	 protocol
	 utils