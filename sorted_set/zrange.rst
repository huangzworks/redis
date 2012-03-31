.. _zrange:

ZRANGE
=======

**ZRANGE key start stop [WITHSCORES]**

返回有序集 ``key`` 中，指定区间内的成员。

其中成员的位置按 ``score`` 值递增(从小到大)来排序。

具有相同 ``score`` 值的成员按字典序(`lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order>`_ )来排列。

如果你需要成员按 ``score`` 值递减(从大到小)来排列，请使用 :ref:`ZREVRANGE` 命令。

| 下标参数 ``start`` 和 ``stop`` 都以 ``0`` 为底，也就是说，以 ``0`` 表示有序集第一个成员，以 ``1`` 表示有序集第二个成员，以此类推。
| 你也可以使用负数下标，以 ``-1`` 表示最后一个成员， ``-2`` 表示倒数第二个成员，以此类推。

| 超出范围的下标并不会引起错误。
| 比如说，当 ``start`` 的值比有序集的最大下标还要大，或是 ``start > stop`` 时， `ZRANGE`_ 命令只是简单地返回一个空列表。
| 另一方面，假如 ``stop`` 参数的值比有序集的最大下标还要大，那么 Redis 将 ``stop`` 当作最大下标来处理。

| 可以通过使用 ``WITHSCORES`` 选项，来让成员和它的 ``score`` 值一并返回，返回列表以 ``value1,score1, ..., valueN,scoreN`` 的格式表示。
| 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(log(N)+M)， ``N`` 为有序集的基数，而 ``M`` 为结果集的基数。

**返回值:**
    指定区间内，带有 ``score`` 值(可选)的有序集成员的列表。

:: 

   redis > ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"

   redis > ZRANGE salary 1 2 WITHSCORES              # 显示有序集下标区间 1 至 2 的成员
   1) "tom"
   2) "5000"
   3) "boss"
   4) "10086"

   redis > ZRANGE salary 0 200000 WITHSCORES         # 测试 end 下标超出最大下标时的情况
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"

   redis > ZRANGE salary 200000 3000000 WITHSCORES   # 测试当给定区间不存在于有序集时的情况 
   (empty list or set)
