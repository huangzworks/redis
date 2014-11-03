.. _zrangebyscore:

ZRANGEBYSCORE
==============

**ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]**

返回有序集 ``key`` 中，所有 ``score`` 值介于 ``min`` 和 ``max`` 之间(包括等于 ``min`` 或 ``max`` )的成员。有序集成员按 ``score`` 值递增(从小到大)次序排列。

具有相同 ``score`` 值的成员按字典序(`lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order>`_)来排列(该属性是有序集提供的，不需要额外的计算)。

可选的 ``LIMIT`` 参数指定返回结果的数量及区间(就像SQL中的 ``SELECT LIMIT offset, count`` )，注意当 ``offset`` 很大时，定位 ``offset`` 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。

| 可选的 ``WITHSCORES`` 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 ``score`` 值一起返回。
| 该选项自 Redis 2.0 版本起可用。

**区间及无限**

``min`` 和 ``max`` 可以是 ``-inf`` 和 ``+inf`` ，这样一来，你就可以在不知道有序集的最低和最高 ``score`` 值的情况下，使用 `ZRANGEBYSCORE`_ 这类命令。

默认情况下，区间的取值使用\ `闭区间 <http://zh.wikipedia.org/wiki/%E5%8D%80%E9%96%93>`_ (小于等于或大于等于)，你也可以通过给参数前增加 ``(`` 符号来使用可选的\ `开区间 <http://zh.wikipedia.org/wiki/%E5%8D%80%E9%96%93>`_ (小于或大于)。

举个例子：

:: 

    ZRANGEBYSCORE zset (1 5

返回所有符合条件 ``1 < score <= 5`` 的成员，而

::

    ZRANGEBYSCORE zset (5 (10

则返回所有符合条件 ``5 < score < 10`` 的成员。

**可用版本：**
    >= 1.0.5

**时间复杂度:**
    O(log(N)+M)， ``N`` 为有序集的基数， ``M`` 为被结果集的基数。

**返回值:**
    指定区间内，带有 ``score`` 值(可选)的有序集成员的列表。

::

    redis> ZADD salary 2500 jack                        # 测试数据
    (integer) 0
    redis> ZADD salary 5000 tom
    (integer) 0
    redis> ZADD salary 12000 peter
    (integer) 0

    redis> ZRANGEBYSCORE salary -inf +inf               # 显示整个有序集
    1) "jack"
    2) "tom"
    3) "peter"

    redis> ZRANGEBYSCORE salary -inf +inf WITHSCORES    # 显示整个有序集及成员的 score 值
    1) "jack"
    2) "2500"
    3) "tom"
    4) "5000"
    5) "peter"
    6) "12000"

    redis> ZRANGEBYSCORE salary -inf 5000 WITHSCORES    # 显示工资 <=5000 的所有成员
    1) "jack"
    2) "2500"
    3) "tom"
    4) "5000"

    redis> ZRANGEBYSCORE salary (5000 400000            # 显示工资大于 5000 小于等于 400000 的成员
    1) "peter"
