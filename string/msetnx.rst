.. _msetnx:

MSETNX
========

.. function:: MSETNX key value [key value ...]

同时设置一个或多个\ ``key-value``\ 对，当且仅当\ ``key``\ 不存在。

即使\ *只有一个*\ \ ``key``\ 已存在，\ `MSETNX`_\ 也会拒绝\ *所有*\ 传入\ ``key``\ 的设置操作

`MSETNX`_\ 是原子性的，因此它可以用作设置多个不同\ ``key``\ 表示不同字段(field)的唯一性逻辑对象(unique logic object)，所有字段要么全被设置，要么全不被设置。

**时间复杂度：**
    O(N)，\ ``N``\ 为要设置的\ ``key``\ 的数量。

**返回值：**
    | 当所有\ ``key``\ 都成功设置，返回\ ``1``\ 。
    | 如果所有key都设置失败(最少有一个\ ``key``\ 已经存在)，那么返回\ ``0``\ 。

::

    # 情况1：对不存在的key进行MSETNX

    redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
    (integer) 1


    # 情况2：对已存在的key进行MSETNX

    redis> MSETNX rmdbs "Sqlite" language "python"  # rmdbs键已经存在，操作失败
    (integer) 0

    redis> EXISTS language  # 因为操作是原子性的，language没有被设置
    (integer) 0

    redis> GET rmdbs  # rmdbs没有被修改
    "MySQL"

    redis> MGET rmdbs nosql key-value-store  
    1) "MySQL"
    2) "MongoDB"
    3) "redis"



