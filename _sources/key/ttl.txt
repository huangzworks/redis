.. _ttl:

TTL
====

**TTL key**

以秒为单位，返回给定 ``key`` 的剩余生存时间(TTL, time to live)。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 当 ``key`` 不存在时，返回 ``-2`` 。
    | 当 ``key`` 存在但没有设置剩余生存时间时，返回 ``-1`` 。
    | 否则，以秒为单位，返回 ``key`` 的剩余生存时间。

.. note:: 在 Redis 2.8 以前，当 ``key`` 不存在，或者 ``key`` 没有设置剩余生存时间时，命令都返回 ``-1`` 。

::

    # 不存在的 key

    redis> FLUSHDB
    OK

    redis> TTL key
    (integer) -2


    # key 存在，但没有设置剩余生存时间 

    redis> SET key value
    OK

    redis> TTL key
    (integer) -1


    # 有剩余生存时间的 key

    redis> EXPIRE key 10086
    (integer) 1

    redis> TTL key
    (integer) 10084
