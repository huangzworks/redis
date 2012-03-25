.. _hget:

HGET
=====
.. function:: HGET key field

返回哈希表\ ``key``\ 中给定域\ ``field``\ 的值。

**时间复杂度：**
    O(1)

**返回值：**
    | 给定域的值。
    | 当给定域不存在或是给定\ ``key``\ 不存在时，返回\ ``nil``\ 。

::

    redis> HSET huangz blog huangz.iteye.com
    (integer) 1

    redis> HGET huangz blog
    "huangz.iteye.com"


