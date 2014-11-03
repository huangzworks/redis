.. _zremrangebyrank:

ZREMRANGEBYRANK
================

**ZREMRANGEBYRANK key start stop**

移除有序集 ``key`` 中，指定排名(rank)区间内的所有成员。

区间分别以下标参数 ``start`` 和 ``stop`` 指出，包含 ``start`` 和 ``stop`` 在内。

| 下标参数 ``start`` 和 ``stop`` 都以 ``0`` 为底，也就是说，以 ``0`` 表示有序集第一个成员，以 ``1`` 表示有序集第二个成员，以此类推。
| 你也可以使用负数下标，以 ``-1`` 表示最后一个成员， ``-2`` 表示倒数第二个成员，以此类推。

**可用版本：**
    >= 2.0.0

**时间复杂度:**
    O(log(N)+M)， ``N`` 为有序集的基数，而 ``M`` 为被移除成员的数量。

**返回值:**
    被移除成员的数量。

::

    redis> ZADD salary 2000 jack
    (integer) 1
    redis> ZADD salary 5000 tom
    (integer) 1
    redis> ZADD salary 3500 peter
    (integer) 1

    redis> ZREMRANGEBYRANK salary 0 1       # 移除下标 0 至 1 区间内的成员
    (integer) 2

    redis> ZRANGE salary 0 -1 WITHSCORES    # 有序集只剩下一个成员
    1) "tom"
    2) "5000"
