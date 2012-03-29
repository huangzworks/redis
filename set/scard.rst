.. _scard:

SCARD
======

**SCARD key**

返回集合\ ``key``\ 的\ **基数**\(集合中元素的数量)。

**时间复杂度:**
    O(1)

**返回值：**
    | 集合的基数。
    | 当\ ``key``\ 不存在时，返回\ ``0``\ 。

::

    redis> SMEMBERS tool
    1) "pc"
    2) "printer"
    3) "phone"

    redis> SCARD tool
    (integer) 3

    redis> SMEMBERS fake_set
    (empty list or set)

    redis> SCARD fake_set
    (integer) 0


