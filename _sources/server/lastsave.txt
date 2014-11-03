.. _lastsave:

LASTSAVE
=========

**LASTSAVE**

返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    一个 UNIX 时间戳。

::

    redis> LASTSAVE
    (integer) 1324043588


