.. _setex:

SETEX
======

**SETEX key seconds value**

将值 ``value`` 关联到 ``key`` ，并将 ``key`` 的生存时间设为 ``seconds`` (以秒为单位)。

如果 ``key`` \ 已经存在， `SETEX`_ 命令将覆写旧值。

这个命令类似于以下两个命令：

::

    SET key value
    EXPIRE key seconds  # 设置生存时间

不同之处是， `SETEX`_ 是一个原子性(atomic)操作，关联值和设置生存时间两个动作会在同一时间内完成，该命令在 Redis 用作缓存时，非常实用。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功时返回 ``OK`` 。
    | 当 ``seconds`` 参数不合法时，返回一个错误。

::

    # 在 key 不存在时进行 SETEX

    redis> SETEX cache_user_id 60 10086
    OK

    redis> GET cache_user_id  # 值
    "10086"
     
    redis> TTL cache_user_id  # 剩余生存时间
    (integer) 49


    # key 已经存在时，SETEX 覆盖旧值

    redis> SET cd "timeless"
    OK

    redis> SETEX cd 3000 "goodbye my love"
    OK

    redis> GET cd
    "goodbye my love"

    redis> TTL cd
    (integer) 2997
