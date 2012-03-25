.. _get:

GET
====

.. function:: GET key 
    
返回\ ``key``\ 所关联的字符串值。

如果\ ``key``\ 不存在则返回特殊值\ ``nil``\ 。

假如\ ``key``\ 储存的值不是字符串类型，返回一个错误，因为\ `GET`_\ 只能用于处理字符串值。

**时间复杂度：**
    O(1)

**返回值：**
    | \ ``key``\ 的值。
    | 如果\ ``key``\ 不存在，返回\ ``nil``\ 。

::

    redis> GET fake_key
    (nil)

    redis> SET animate "anohana"
    OK

    redis> GET animate
    "anohana"


