.. _pttl:

PTTL
======

**PTTL key**

这个命令类似于 :ref:`TTL` 命令，但它以毫秒为单位返回 ``key`` 的剩余生存时间，而不是像 :ref:`TTL` 命令那样，以秒为单位。

**可用版本：**
    >= 2.6.0

**复杂度：**
    O(1)

**返回值：**
    | 当 ``key`` 不存在时，返回 ``-2`` 。
    | 当 ``key`` 存在但没有设置剩余生存时间时，返回 ``-1`` 。
    | 否则，以毫秒为单位，返回 ``key`` 的剩余生存时间。

.. note:: 在 Redis 2.8 以前，当 ``key`` 不存在，或者 ``key`` 没有设置剩余生存时间时，命令都返回 ``-1`` 。

::

    # 不存在的 key

    redis> FLUSHDB
    OK

    redis> PTTL key
    (integer) -2


    # key 存在，但没有设置剩余生存时间 

    redis> SET key value
    OK

    redis> PTTL key
    (integer) -1


    # 有剩余生存时间的 key

    redis> PEXPIRE key 10086
    (integer) 1

    redis> PTTL key
    (integer) 6179
