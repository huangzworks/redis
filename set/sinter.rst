.. _sinter:

SINTER
========

**SINTER key [key ...]**

返回一个集合的全部成员，该集合是所有给定集合的交集。

不存在的 ``key`` 被视为空集。

当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(N * M)， ``N`` 为给定集合当中基数最小的集合， ``M`` 为给定集合的个数。

**返回值:**
    交集成员的列表。

::

    redis> SMEMBERS group_1
    1) "LI LEI"
    2) "TOM"
    3) "JACK"

    redis> SMEMBERS group_2
    1) "HAN MEIMEI"
    2) "JACK" 

    redis> SINTER group_1 group_2
    1) "JACK"
