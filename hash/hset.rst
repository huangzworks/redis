.. _hset:

HSET
=====

**HSET key field value**

将哈希表 ``key`` 中的域 ``field`` 的值设为 ``value`` 。

如果 ``key`` 不存在，一个新的哈希表被创建并进行 `HSET`_ 操作。

如果域 ``field`` 已经存在于哈希表中，旧值将被覆盖。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 如果 ``field`` 是哈希表中的一个新建域，并且值设置成功，返回 ``1`` 。
    | 如果哈希表中域 ``field`` 已经存在且旧值已被新值覆盖，返回 ``0`` 。

::

    redis> HSET website google "www.g.cn"       # 设置一个新域
    (integer) 1

    redis> HSET website google "www.google.com" # 覆盖一个旧域
    (integer) 0
