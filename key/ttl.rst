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
    | 当 ``key`` 不存在或没有设置生存时间时，返回 ``-1``  。
    | 否则，返回 ``key`` 的剩余生存时间(以秒为单位)。

::

    # 带 TTL 的key

    redis> SET name "redis"
    OK

    redis> EXPIRE name 30
    (integer) 1

    redis> TTL name
    (integer) 27

    redis> TTL name
    (integer) 13

    redis> TTL name  # 过期返回 -1
    (integer) -1

    redis> GET name  # 并且 key 被删除
    (nil)

    # 不带TTL的key

    redis> SET site wikipedia.org   
    OK

    redis> TTL wikipedia.org
    (integer) -1


    # 不存在的key

    redis> EXISTS not_exists_key
    (integer) 0

    redis> TTL not_exists_key
    (integer) -1
