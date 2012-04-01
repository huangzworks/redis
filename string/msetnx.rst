.. _msetnx:

MSETNX
========

**MSETNX key value [key value ...]**

同时设置一个或多个 ``key-value`` 对，当且仅当所有给定 ``key`` 都不存在。

即使只有一个给定 ``key`` 已存在， `MSETNX`_ 也会拒绝执行所有给定 ``key`` 的设置操作。

`MSETNX`_ 是原子性的，因此它可以用作设置多个不同 ``key`` 表示不同字段(field)的唯一性逻辑对象(unique logic object)，所有字段要么全被设置，要么全不被设置。

**可用版本：**
    >= 1.0.1

**时间复杂度：**
    O(N)， ``N`` 为要设置的 ``key`` 的数量。

**返回值：**
    | 当所有 ``key`` 都成功设置，返回 ``1`` 。
    | 如果所有给定 ``key`` 都设置失败(至少有一个 ``key`` 已经存在)，那么返回 ``0`` 。

::

    # 对不存在的 key 进行 MSETNX

    redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
    (integer) 1

    redis> MGET rmdbs nosql key-value-store  
    1) "MySQL"
    2) "MongoDB"
    3) "redis"


    # MSET 的给定 key 当中有已存在的 key

    redis> MSETNX rmdbs "Sqlite" language "python"  # rmdbs 键已经存在，操作失败
    (integer) 0

    redis> EXISTS language                          # 因为 MSET 是原子性操作，language 没有被设置
    (integer) 0

    redis> GET rmdbs                                # rmdbs 也没有被修改
    "MySQL"
