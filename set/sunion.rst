.. _sunion:

SUNION
=======

**SUNION key [key ...]**

返回一个集合的全部成员，该集合是所有给定集合的并集。

不存在的 ``key`` 被视为空集。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(N)， ``N`` 是所有给定集合的成员数量之和。

**返回值:**
    并集成员的列表。

::

    redis> SMEMBERS songs
    1) "Billie Jean"

    redis> SMEMBERS my_songs
    1) "Believe Me"

    redis> SUNION songs my_songs
    1) "Billie Jean"
    2) "Believe Me"
