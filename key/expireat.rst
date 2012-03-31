.. _expireat:

EXPIREAT
========

**EXPIREAT key timestamp**

`EXPIREAT`_ 的作用和 :doc:`expire` 类似，都用于为 ``key`` 设置生存时间。

不同在于 `EXPIREAT`_ 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。

**可用版本：**
    >= 1.2.0

**时间复杂度：**
    O(1)

**返回值：**
    | 如果生存时间设置成功，返回 ``1`` 。
    | 当 ``key`` 不存在或没办法设置生存时间，返回 ``0`` 。

::

    redis> SET cache www.google.com
    OK

    redis> EXPIREAT cache 1355292000     # 这个 key 将在 2012.12.12 过期
    (integer) 1

    redis> TTL cache
    (integer) 45081860
