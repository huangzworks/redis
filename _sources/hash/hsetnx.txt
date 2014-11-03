.. _hsetnx:

HSETNX
=======

**HSETNX key field value**

将哈希表 ``key`` 中的域 ``field`` 的值设置为 ``value`` ，当且仅当域 ``field`` 不存在。

若域 ``field`` 已经存在，该操作无效。

如果 ``key`` 不存在，一个新哈希表被创建并执行 `HSETNX`_ 命令。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功，返回 ``1`` 。
    | 如果给定域已经存在且没有操作被执行，返回 ``0`` 。

::

    redis> HSETNX nosql key-value-store redis
    (integer) 1

    redis> HSETNX nosql key-value-store redis       # 操作无效，域 key-value-store 已存在
    (integer) 0
