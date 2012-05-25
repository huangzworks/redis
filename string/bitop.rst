.. _bitop:

BITOP
=======

**BITOP destkey operation key [key ...]**

对一个或多个保存二进制位的字符串 ``key`` 进行位元操作，并将结果保存到 ``destkey`` 上。

``operation`` 可以是 ``AND`` 、 ``OR`` 、 ``NOT`` 、 ``XOR`` 这四种操作中的任意一种：

- ``BITOP AND destkey key [key ...]`` ，对一个或多个 ``key`` 求逻辑并，并将结果保存到 ``destkey`` 。

- ``BITOP OR destkey key [key ...]`` ，对一个或多个 ``key`` 求逻辑或，并将结果保存到 ``destkey`` 。

- ``BITOP XOR destkey key [key ...]`` ，对一个或多个 ``key`` 求逻辑异或，并将结果保存到 ``destkey`` 。

- ``BITOP NOT destkey key`` ，对给定 ``key`` 求逻辑非，并将结果保存到 ``destkey`` 。

除了 ``NOT`` 操作之外，其他操作都可以接受一个或多个 ``key`` 作为输入。

**处理不同长度的字符串**

当 `BITOP`_ 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 ``0`` 。

空的 ``key`` 也被看作包含 ``0`` 的字符串序列。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(N)

**返回值：**
    保存到 ``destkey`` 的字符串的长度，和输入 ``key`` 中最长的字符串相等。

.. note:: `BITOP`_ 的复杂度为 O(N) ，当处理大型矩阵(matrix)或者进行大数据量的统计时，最好将任务指派到附属节点(slave)进行，避免阻塞主节点。

::

    redis> SET bits-1 1111
    OK

    redis> SET bits-2 1001
    OK

    redis> BITOP AND and-bits bits-1 bits-2
    (integer) 4

    redis> GET and-bits
    "1001"

    redis> BITOP XOR xor-bits bits-1 bits-2
    (integer) 4

    redis> GET xor-bits                         # 0110
    "\x00\x01\x01\x00"

    redis> BITOP NOT negation-bits bits-2
    (integer) 4

    redis> GET negation-bits                    # 0110
    "\xce\xcf\xcf\xce"


模式：使用位图实现实时矩阵运算
----------------------------------

请参见 :doc:`bitcount` 命令。
