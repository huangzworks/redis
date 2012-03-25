.. _lastsave:

LASTSAVE
=========

**LASTSAVE**

返回最近一次 Redis 成功执行保存操作的时间点( `SAVE`_ 、 `BGSAVE`_ 等)，以 UNIX 时间戳格式表示。

**时间复杂度：**
    O(1)

**返回值：**
    一个 UNIX 时间戳。

::

    redis> LASTSAVE
    (integer) 1324043588


