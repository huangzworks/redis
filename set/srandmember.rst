.. _srandmember:

SRANDMEMBER
============

**SRANDMEMBER key**

返回集合中的一个随机元素。

该操作和 :ref:`SPOP` 相似，但 :ref:`SPOP` 将随机元素从集合中移除并返回，而 `SRANDMEMBER`_ 则仅仅返回随机元素，而不对集合进行任何改动。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(1)

**返回值:**
    被选中的随机元素。
    当 ``key`` 不存在或 ``key`` 是空集时，返回 ``nil`` 。

::

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"

    redis> SRANDMEMBER joe's_movies
    "Fast Five"

    redis> SMEMBERS joe's_movies    # 集合中的元素不变
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"
