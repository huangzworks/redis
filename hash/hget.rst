.. _hget:

HGET
=====

**HGET key field**

返回哈希表 ``key`` 中给定域 ``field`` 的值。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 给定域的值。
    | 当给定域不存在或是给定 ``key`` 不存在时，返回 ``nil`` 。

::

    # 域存在

    redis> HSET site redis redis.com
    (integer) 1

    redis> HGET site redis
    "redis.com"

    
    # 域不存在

    redis> HGET site mysql
    (nil)
