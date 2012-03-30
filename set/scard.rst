.. _scard:

SCARD
======

**SCARD key**

返回集合 ``key`` 的基数(集合中元素的数量)。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(1)

**返回值：**
    | 集合的基数。
    | 当 ``key`` 不存在时，返回 ``0`` 。

::

    redis> SADD tool pc printer phone
    (integer) 3

    redis> SCARD tool   # 非空集合
    (integer) 3

    redis> DEL tool
    (integer) 1

    redis> SCARD tool   # 空集合
    (integer) 0
