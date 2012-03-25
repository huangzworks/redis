.. _ttl:

TTL
====

.. function:: TTL key

返回给定\ ``key``\ 的剩余生存时间(time to live)(以秒为单位)。

**时间复杂度：**
    O(1)

**返回值：**
    | \ ``key``\ 的剩余生存时间(以秒为单位)。
    | 当\ ``key``\ 不存在或没有设置生存时间时，返回\ ``-1``\  。

::

    # 情况1：带TTL的key

    redis> set name "huangz"    # 设置一个key
    OK

    redis> expire name 30  # 设置生存时间为30秒
    (integer) 1

    redis> get name
    "huangz"

    redis> ttl name
    (integer) 25

    redis> ttl name # 30秒过去，name过期
    (integer) -1

    redis> get name # 过期的key将被删除
    (nil)


    # 情况2：不带TTL的key

    redis> SET site wikipedia.org   
    OK

    redis> TTL wikipedia.org
    (integer) -1


    # 情况3：不存在的key

    redis> EXISTS not_exists_key
    (integer) 0

    redis> TTL not_exists_key
    (integer) -1


