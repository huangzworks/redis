.. _zrevrank:

ZREVRANK
=========

.. function:: ZREVRANK key member

返回有序集\ ``key``\ 中成员\ ``member``\ 的排名。其中有序集成员按\ ``score``\ 值递减(从大到小)排序。

排名以\ ``0``\ 为底，也就是说，\ ``score``\ 值最大的成员排名为\ ``0``\ 。

使用\ `ZRANK`_\ 命令可以获得成员按\ ``score``\ 值递增(从小到大)排列的排名。

**时间复杂度:**
    O(log(N))

**返回值:**
    | 如果\ ``member``\ 是有序集\ ``key``\ 的成员，返回\ ``member``\ 的排名。
    | 如果\ ``member``\ 不是有序集\ ``key``\ 的成员，返回\ ``nil``\ 。

::

    redis> ZADD salary 2000 jack
    (integer) 1
    redis> ZADD salary 5000 tom
    (integer) 1
    redis> ZADD salary 3500 peter
    (integer) 1

    redis> ZREVRANK salary peter # peter 的工资排第二
    (integer) 1
    redis> ZREVRANK salary tom   # tom 的工资最高
    (integer) 0


