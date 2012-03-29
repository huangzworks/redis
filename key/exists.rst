.. _exists:

EXISTS
======

**EXISTS key**

检查给定 ``key`` 是否存在。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    若 ``key`` 存在，返回 ``1`` ，否则返回 ``0`` 。

::

    redis> SET db "redis"
    OK

    redis> EXISTS db
    (integer) 1

    redis> DEL db
    (integer) 1

    redis> EXISTS db
    (integer) 0
