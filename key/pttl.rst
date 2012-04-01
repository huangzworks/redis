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
    | 如果 ``key`` 不存在，返回 ``-1`` 。
    | 否则，返回以毫秒为单位表示的 ``key`` 的剩余生存时间。

::

    redis> SET mykey "Hello"
    OK

    redis> EXPIRE mykey 1
    (integer) 1

    redis> PTTL mykey
    (integer) 1000


    # 对不存在的 key 返回 -1

    redis> EXISTS some_key
    (integer) 0

    redis> PTTL some_key
    (integer) -1

