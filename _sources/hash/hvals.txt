.. _hvals:

HVALS
======

**HVALS key**

返回哈希表 ``key`` 中所有域的值。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(N)， ``N`` 为哈希表的大小。

**返回值：**
    | 一个包含哈希表中所有值的表。
    | 当 ``key`` 不存在时，返回一个空表。

::

    # 非空哈希表

    redis> HMSET website google www.google.com yahoo www.yahoo.com 
    OK

    redis> HVALS website
    1) "www.google.com"
    2) "www.yahoo.com"


    # 空哈希表/不存在的key

    redis> EXISTS not_exists
    (integer) 0

    redis> HVALS not_exists
    (empty list or set)
