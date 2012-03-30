.. _spop:

SPOP
=====

**SPOP key**

移除并返回集合中的一个随机元素。

如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用 :ref:`SRANDMEMBER` 命令。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(1)

**返回值:**
    | 被移除的随机元素。
    | 当 ``key`` 不存在或 ``key`` 是空集时，返回 ``nil`` 。

::

    redis> SMEMBERS db
    1) "MySQL"
    2) "MongoDB"
    3) "Redis"

    redis> SPOP db
    "Redis"

    redis> SMEMBERS db
    1) "MySQL"
    2) "MongoDB"

    redis> SPOP db
    "MySQL"

    redis> SMEMBERS db
    1) "MongoDB"
