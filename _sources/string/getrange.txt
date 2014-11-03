.. _getrange:

GETRANGE
=========

**GETRANGE key start end**

返回 ``key`` 中字符串值的子字符串，字符串的截取范围由 ``start`` 和 ``end`` 两个偏移量决定(包括 ``start`` 和 ``end`` 在内)。

负数偏移量表示从字符串最后开始计数， ``-1`` 表示最后一个字符， ``-2`` 表示倒数第二个，以此类推。

`GETRANGE`_ 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。

.. note::
    在 <= 2.0 的版本里，GETRANGE 被叫作 SUBSTR。

**可用版本：**
    >= 2.4.0

**时间复杂度：**
    | O(N)， ``N`` 为要返回的字符串的长度。
    | 复杂度最终由字符串的返回值长度决定，但因为从已有字符串中取出子字符串的操作非常廉价(cheap)，所以对于长度不大的字符串，该操作的复杂度也可看作O(1)。

**返回值：**
    截取得出的子字符串。

::

    redis> SET greeting "hello, my friend"
    OK

    redis> GETRANGE greeting 0 4          # 返回索引0-4的字符，包括4。
    "hello"

    redis> GETRANGE greeting -1 -5        # 不支持回绕操作
    ""

    redis> GETRANGE greeting -3 -1        # 负数索引
    "end"

    redis> GETRANGE greeting 0 -1         # 从第一个到最后一个
    "hello, my friend"

    redis> GETRANGE greeting 0 1008611    # 值域范围不超过实际字符串，超过部分自动被符略
    "hello, my friend"
