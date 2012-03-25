.. _exists:

EXISTS
======

.. function:: EXISTS key

检查给定\ ``key``\ 是否存在。

**时间复杂度：**
    O(1)

**返回值：**
    若\ ``key``\ 存在，返回\ ``1``\ ，否则返回\ ``0``\ 。

::

    redis> set db "redis"
    OK

    redis> exists db  # key存在
    (integer) 1

    redis> del db   # 删除key
    (integer) 1

    redis> exists db  # key不存在
    (integer) 0
