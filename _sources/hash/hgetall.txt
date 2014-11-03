.. _hgetall:

HGETALL
=======

**HGETALL key**

返回哈希表 ``key`` 中，所有的域和值。

在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(N)， ``N`` 为哈希表的大小。

**返回值：**
    | 以列表形式返回哈希表的域和域的值。
    | 若 ``key`` 不存在，返回空列表。

::

    redis> HSET people jack "Jack Sparrow"
    (integer) 1

    redis> HSET people gump "Forrest Gump"
    (integer) 1

    redis> HGETALL people
    1) "jack"          # 域
    2) "Jack Sparrow"  # 值
    3) "gump"
    4) "Forrest Gump"
