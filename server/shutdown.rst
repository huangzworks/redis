.. _shutdown:

SHUTDOWN
=========

**SHUTDOWN**

`SHUTDOWN`_ 命令执行以下操作：

- 停止所有客户端
- 如果有最少一个保存点在等待，执行 `SAVE`_ 命令
- 如果 AOF 选项被打开，更新 AOF 文件
- 服务器关闭

如果持久化被打开的话， `SHUTDOWN`_ 命令会保证服务器正常关闭而\ *不*\ 丢失任何数据。

假如只是单纯地执行 `SAVE`_ 命令，然后再执行 :ref:`quit` 命令，则没有这一保证 —— 因为在执行 `SAVE`_ 之后、执行 :ref:`quit` 之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 :ref:`quit` 就会造成数据丢失。

**时间复杂度：**
    不明确

**返回值：**
    | 执行失败时返回错误。
    | 执行成功时不返回任何信息，服务器和客户端的连接断开，客户端自动退出。

::

    redis> PING
    PONG

    redis> SHUTDOWN  

    [huangz@mypad]$ 

    [huangz@mypad]$ redis
    Could not connect to Redis at: Connection refused
    not connected> 



