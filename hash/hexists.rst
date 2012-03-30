.. _hexists:

HEXISTS
========

**HEXISTS key field**

查看哈希表 ``key`` 中，给定域 ``field`` 是否存在。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 如果哈希表含有给定域，返回 ``1`` 。
    | 如果哈希表不含有给定域，或 ``key`` 不存在，返回 ``0`` 。

::

    redis> HEXISTS phone myphone
    (integer) 0

    redis> HSET phone myphone nokia-1110
    (integer) 1

    redis> HEXISTS phone myphone
    (integer) 1
