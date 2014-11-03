.. _zrevrangebyscore:

ZREVRANGEBYSCORE
=================

**ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]**

返回有序集 ``key`` 中， ``score`` 值介于 ``max`` 和 ``min`` 之间(默认包括等于 ``max`` 或 ``min`` )的所有的成员。有序集成员按 ``score`` 值递减(从大到小)的次序排列。

具有相同 ``score`` 值的成员按字典序的逆序(`reverse lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order>`_ )排列。

除了成员按 ``score`` 值递减的次序排列这一点外， `ZREVRANGEBYSCORE`_ 命令的其他方面和 :ref:`ZRANGEBYSCORE` 命令一样。

**可用版本：**
    >= 2.2.0

**时间复杂度:**
    O(log(N)+M)， ``N`` 为有序集的基数， ``M`` 为结果集的基数。

**返回值:**
    指定区间内，带有 ``score`` 值(可选)的有序集成员的列表。

::

    redis > ZADD salary 10086 jack
    (integer) 1
    redis > ZADD salary 5000 tom
    (integer) 1
    redis > ZADD salary 7500 peter
    (integer) 1
    redis > ZADD salary 3500 joe
    (integer) 1

    redis > ZREVRANGEBYSCORE salary +inf -inf   # 逆序排列所有成员
    1) "jack"
    2) "peter"
    3) "tom"
    4) "joe"

    redis > ZREVRANGEBYSCORE salary 10000 2000  # 逆序排列薪水介于 10000 和 2000 之间的成员
    1) "peter"
    2) "tom"
    3) "joe"
