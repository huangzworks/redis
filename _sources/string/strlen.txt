.. _strlen:

STRLEN
=======

**STRLEN key**

返回 ``key`` 所储存的字符串值的长度。

当 ``key`` 储存的不是字符串值时，返回一个错误。

**可用版本：**
    >= 2.2.0

**复杂度：**
    O(1)

**返回值：**
    | 字符串值的长度。
    | 当  ``key`` 不存在时，返回 ``0`` 。

::

    # 获取字符串的长度

    redis> SET mykey "Hello world"
    OK

    redis> STRLEN mykey
    (integer) 11


    # 不存在的 key 长度为 0

    redis> STRLEN nonexisting
    (integer) 0
