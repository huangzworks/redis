.. _expireat:

EXPIREAT
========

.. function:: EXPIREAT key timestamp

\ `EXPIREAT`_\ 的作用和\ `EXPIRE`_\ 一样，都用于为\ ``key``\ 设置生存时间。

不同在于\ `EXPIREAT`_\ 命令接受的时间参数是\ *UNIX时间戳*\ (unix timestamp)。

**时间复杂度：**
    O(1)

**返回值：**
    | 如果生存时间设置成功，返回\ ``1``\ 。
    | 当\ ``key``\ 不存在或没办法设置生存时间，返回\ ``0``\ 。

::

    redis> SET cache www.google.com
    OK

    redis> EXPIREAT cache 1355292000 # 这个key将在2012.12.12过期
    (integer) 1

    redis> TTL cache
    (integer) 45081860


