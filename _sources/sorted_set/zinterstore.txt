.. _zinterstore:

ZINTERSTORE
=============

**ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]**

计算给定的一个或多个有序集的交集，其中给定 ``key`` 的数量必须以 ``numkeys`` 参数指定，并将该交集(结果集)储存到 ``destination`` 。

默认情况下，结果集中某个成员的 ``score`` 值是所有给定集下该成员 ``score`` 值之和.

关于 ``WEIGHTS`` 和 ``AGGREGATE`` 选项的描述，参见 :ref:`ZUNIONSTORE` 命令。

**可用版本：**
    >= 2.0.0

**时间复杂度:**
    O(N*K)+O(M*log(M))， ``N`` 为给定 ``key`` 中基数最小的有序集， ``K`` 为给定有序集的数量， ``M`` 为结果集的基数。

**返回值:**
    保存到 ``destination`` 的结果集的基数。

::
    
    redis > ZADD mid_test 70 "Li Lei"
    (integer) 1
    redis > ZADD mid_test 70 "Han Meimei"
    (integer) 1
    redis > ZADD mid_test 99.5 "Tom"
    (integer) 1

    redis > ZADD fin_test 88 "Li Lei"
    (integer) 1
    redis > ZADD fin_test 75 "Han Meimei"
    (integer) 1
    redis > ZADD fin_test 99.5 "Tom"
    (integer) 1

    redis > ZINTERSTORE sum_point 2 mid_test fin_test
    (integer) 3

    redis > ZRANGE sum_point 0 -1 WITHSCORES     # 显示有序集内所有成员及其 score 值
    1) "Han Meimei"
    2) "145"
    3) "Li Lei"
    4) "158"
    5) "Tom"
    6) "199"
