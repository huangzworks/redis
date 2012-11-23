.. _client_kill:

CLIENT KILL
===============

**CLIENT KILL ip:port**

关闭地址为 ``ip:port`` 的客户端。

``ip:port`` 应该和 :ref:`client_list` 命令输出的其中一行匹配。

因为 Redis 使用单线程设计，所以当 Redis 正在执行命令的时候，不会有客户端被断开连接。

如果要被断开连接的客户端正在执行命令，那么当这个命令执行之后，在发送下一个命令的时候，它就会收到一个网络错误，告知它自身的连接已被关闭。

**可用版本**
    >= 2.4.0

**时间复杂度**
    O(N) ， N 为已连接的客户端数量。

**返回值**
    当指定的客户端存在，且被成功关闭时，返回 ``OK`` 。

::

    # 列出所有已连接客户端

    redis 127.0.0.1:6379> CLIENT LIST
    addr=127.0.0.1:43501 fd=5 age=10 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client

    # 杀死当前客户端的连接

    redis 127.0.0.1:6379> CLIENT KILL 127.0.0.1:43501
    OK

    # 之前的连接已经被关闭，CLI 客户端又重新建立了连接
    # 之前的端口是 43501 ，现在是 43504

    redis 127.0.0.1:6379> CLIENT LIST
    addr=127.0.0.1:43504 fd=5 age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
