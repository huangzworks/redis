.. _client_kill:

CLIENT KILL
===============

**CLIENT KILL ip:port**

关闭地址为 ``ip:port`` 的客户端。

``ip:port`` 应该和 :ref:`client_kill`` 命令输出的其中一行匹配。

因为 Redis 使用单线程设计，所以不可能在客户端执行命令的过程中将其杀死。从客户端的角度来看，在执行命令的过程中，连接是不可能被关闭的。然而，当这个客户端在发送下个命令的时候，就会接到一个网络错误，通知它的连接已被断开。

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
