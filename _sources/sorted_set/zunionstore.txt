.. _zunionstore:

ZUNIONSTORE
============

**ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]**

计算给定的一个或多个有序集的并集，其中给定 ``key`` 的数量必须以 ``numkeys`` 参数指定，并将该并集(结果集)储存到 ``destination`` 。

默认情况下，结果集中某个成员的 ``score`` 值是所有给定集下该成员 ``score`` 值之 *和* 。

**WEIGHTS**

使用 ``WEIGHTS`` 选项，你可以为 *每个* 给定有序集 *分别* 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 ``score`` 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。

如果没有指定 ``WEIGHTS`` 选项，乘法因子默认设置为 ``1`` 。

**AGGREGATE**

使用 ``AGGREGATE`` 选项，你可以指定并集的结果集的聚合方式。

默认使用的参数 ``SUM`` ，可以将所有集合中某个成员的 ``score`` 值之 *和* 作为结果集中该成员的 ``score`` 值；使用参数 ``MIN`` ，可以将所有集合中某个成员的 *最小*  ``score`` 值作为结果集中该成员的 ``score`` 值；而参数 ``MAX`` 则是将所有集合中某个成员的 *最大*  ``score`` 值作为结果集中该成员的 ``score`` 值。

**可用版本：**
    >= 2.0.0

**时间复杂度:**
    O(N)+O(M log(M))， ``N`` 为给定有序集基数的总和， ``M`` 为结果集的基数。

**返回值:**
    保存到 ``destination`` 的结果集的基数。

::

    redis> ZRANGE programmer 0 -1 WITHSCORES
    1) "peter"
    2) "2000"
    3) "jack"
    4) "3500"
    5) "tom"
    6) "5000"

    redis> ZRANGE manager 0 -1 WITHSCORES
    1) "herry"
    2) "2000"
    3) "mary"
    4) "3500"
    5) "bob"
    6) "4000"

    redis> ZUNIONSTORE salary 2 programmer manager WEIGHTS 1 3   # 公司决定加薪。。。除了程序员。。。
    (integer) 6

    redis> ZRANGE salary 0 -1 WITHSCORES
    1) "peter"
    2) "2000"
    3) "jack"
    4) "3500"
    5) "tom"
    6) "5000"
    7) "herry"
    8) "6000"
    9) "mary"
    10) "10500"
    11) "bob"
    12) "12000"
