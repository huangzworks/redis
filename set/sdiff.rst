.. _sdiff:

SDIFF
======

.. function:: SDIFF key [key ...]

返回一个集合的全部成员，该集合是所有给定集合的\ *差集* \。

不存在的\ ``key``\ 被视为空集。

**时间复杂度:**
    O(N)，\ ``N``\ 是所有给定集合的成员数量之和。

**返回值:**
    交集成员的列表。

::

    redis> SMEMBERS peter's_movies
    1) "bet man"
    2) "start war"
    3) "2012"   # <-

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"   # <-

    redis> SDIFF peter's_movies joe's_movies
    1) "bet man"
    2) "start war"


