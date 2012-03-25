.. _bgrewriteaof:

BGREWRITEAOF
=============

**BGREWRITEAOF**

异步(Asynchronously)重写 AOF 文件以反应当前数据库的状态。

即使 `BGREWRITEAOF`_ 命令执行失败，旧 AOF 文件中的数据也不会因此丢失或改变。

请移步 `持久化文档 <http://redis.io/topics/persistence>`_ 查看更多相关细节。

**时间复杂度：**
    O(N)， ``N`` 为要追加到 AOF 文件中的数据数量。

**返回值：**
    反馈信息。

::
    
    redis> BGREWRITEAOF
    Background append only file rewriting started


