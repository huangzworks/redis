.. _shutdown:

SHUTDOWN
=========

**SHUTDOWN**

`SHUTDOWN`_ 命令执行以下操作：

- 停止所有客户端
- 如果有至少一个保存点在等待，执行 :ref:`SAVE` 命令
- 如果 AOF 选项被打开，更新 AOF 文件
- 关闭 redis 服务器(server)

如果持久化被打开的话， `SHUTDOWN`_ 命令会保证服务器正常关闭而不丢失任何数据。

另一方面，假如只是单纯地执行 :ref:`SAVE` 命令，然后再执行 :ref:`QUIT` 命令，则没有这一保证 —— 因为在执行 :ref:`SAVE` 之后、执行 :ref:`QUIT` 之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 :ref:`QUIT` 就会造成数据丢失。

**SAVE 和 NOSAVE 修饰符**

通过使用可选的修饰符，可以修改 `SHUTDOWN`_ 命令的表现。比如说：

- 执行 ``SHUTDOWN SAVE`` 会强制让数据库执行保存操作，即使没有设定(configure)保存点
- 执行 ``SHUTDOWN NOSAVE`` 会阻止数据库执行保存操作，即使已经设定有一个或多个保存点(你可以将这一用法看作是强制停止服务器的一个假想的 ABORT 命令)

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    不明确

**返回值：**
    | 执行失败时返回错误。
    | 执行成功时不返回任何信息，服务器和客户端的连接断开，客户端自动退出。

::

    redis> PING
    PONG

    redis> SHUTDOWN  

    $ 

    $ redis
    Could not connect to Redis at: Connection refused
    not connected> 
