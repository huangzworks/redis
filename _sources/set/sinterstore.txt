.. _sinterstore:

SINTERSTORE
============

**SINTERSTORE destination key [key ...]**

这个命令类似于 :ref:`SINTER` 命令，但它将结果保存到 ``destination`` 集合，而不是简单地返回结果集。

如果 ``destination`` 集合已经存在，则将其覆盖。

``destination`` 可以是 ``key`` 本身。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(N * M)， ``N`` 为给定集合当中基数最小的集合， ``M`` 为给定集合的个数。

**返回值:**
    结果集中的成员数量。

::

    redis> SMEMBERS songs
    1) "good bye joe"
    2) "hello,peter"

    redis> SMEMBERS my_songs
    1) "good bye joe"
    2) "falling"

    redis> SINTERSTORE song_interset songs my_songs
    (integer) 1

    redis> SMEMBERS song_interset
    1) "good bye joe"
