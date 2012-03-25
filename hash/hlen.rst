.. _hlen:

HLEN
======

.. function:: HLEN key

返回哈希表\ ``key``\ 中域的数量。

**时间复杂度：**
    O(1)

**返回值：**
    | 哈希表中域的数量。
    | 当\ ``key``\ 不存在时，返回\ ``0``\ 。

::

    redis> HSET hash_name jack "Jack Sparrow"
    (integer) 1

    redis> HSET hash_name gump "Forrest Gump"
    (integer) 1

    redis> HLEN hash_name
    (integer) 2



