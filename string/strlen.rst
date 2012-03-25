.. _strlen:

STRLEN
=======

.. function:: STRLEN key

返回\ ``key``\ 所储存的字符串值的长度。

当\ ``key``\ 储存的不是字符串值时，返回一个错误。

复杂度：
    O(1)

返回值：
    | 字符串值的长度。
    | 当 \ ``key``\ 不存在时，返回\ ``0``\ 。

::

    redis> SET mykey "Hello world"
    OK

    redis> STRLEN mykey
    (integer) 11

    redis> STRLEN nonexisting # 不存在的key长度视为0
    (integer) 0


