.. _flushdb:

FLUSHDB
=========

**FLUSHDB**

清空\ *当前*\ 数据库中的所有 key 。

此命令从不失败。

**时间复杂度：**
    O(1)

**返回值：**
    总是返回 ``OK`` 。

::

    redis> DBSIZE    # 清空前的 key 数量
    (integer) 4

    redis> FLUSHDB
    OK

    redis> DBSIZE    # 清空后的 key 数量
    (integer) 0


