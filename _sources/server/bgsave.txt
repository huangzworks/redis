.. _bgsave:

BGSAVE
=======

在后台异步(Asynchronously)保存当前数据库的数据到磁盘。

`BGSAVE`_ 命令执行之后立即返回 ``OK`` ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

客户端可以通过 :ref:`LASTSAVE` 命令查看相关信息，判断 `BGSAVE`_ 命令是否执行成功。

请移步 `持久化文档 <http://redis.io/topics/persistence>`_ 查看更多相关细节。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(N)， ``N`` 为要保存到数据库中的 key 的数量。

**返回值：**
    反馈信息。

::

    redis> BGSAVE
    Background saving started
