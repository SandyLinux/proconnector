9.12 类 constants.RefreshOption
==================================

这个类定义了各个刷新操作。

- RefreshOption.GRANT   - 刷新grant tables,像FLUSH PRIVILEGES
- RefreshOption.LOG     - 刷新日志，像FLUSH LOGS
- RefreshOption.TABLES  - 刷新表缓存，像FLUSH TABLES
- RefreshOption.HOSTS   - 刷新host缓存，像FLUSH HOSTS
- RefreshOption.STATUS  - 刷新状态变量，像FLUSH STATUS
- RefreshOption.THREADS - 刷新线程缓存
- RefreshOption.SLAVE   - 在一个从服务器上，重置主服务器信息并且重启从服务器，像RESET SLAVE
- RefreshOption.MASTER  - 在一个主服务器上，移除二进制日志索引中列出的二进制日志文件并且截短这个索引文件，像RESET MASTER