.. _zrevrange:

ZREVRANGE
===========

**ZREVRANGE key start stop [WITHSCORES]**

返回有序集 ``key`` 中，指定区间内的成员。

| 其中成员的位置按 ``score`` 值递减(从大到小)来排列。
| 具有相同 ``score`` 值的成员按字典序的逆序(`reverse lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order#Reverse_lexicographic_order>`_)排列。

除了成员按 ``score`` 值递减的次序排列这一点外， `ZREVRANGE`_ 命令的其他方面和 :ref:`ZRANGE` 命令一样。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(log(N)+M)， ``N`` 为有序集的基数，而 ``M`` 为结果集的基数。

**返回值:**
    指定区间内，带有 ``score`` 值(可选)的有序集成员的列表。

::

    redis> ZRANGE salary 0 -1 WITHSCORES        # 递增排列
    1) "peter"
    2) "3500"
    3) "tom"
    4) "4000"
    5) "jack"
    6) "5000"

    redis> ZREVRANGE salary 0 -1 WITHSCORES     # 递减排列
    1) "jack"
    2) "5000"
    3) "tom"
    4) "4000"
    5) "peter"
    6) "3500"
