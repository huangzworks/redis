.. _bitop:

BITOP
=======

**BITOP operation destkey key [key ...]**

对一个或多个保存二进制位的字符串 ``key`` 进行位元操作，并将结果保存到 ``destkey`` 上。

``operation`` 可以是 ``AND`` 、 ``OR`` 、 ``NOT`` 、 ``XOR`` 这四种操作中的任意一种：

- ``BITOP AND destkey key [key ...]`` ，对一个或多个 ``key`` 求逻辑并，并将结果保存到 ``destkey`` 。

- ``BITOP OR destkey key [key ...]`` ，对一个或多个 ``key`` 求逻辑或，并将结果保存到 ``destkey`` 。

- ``BITOP XOR destkey key [key ...]`` ，对一个或多个 ``key`` 求逻辑异或，并将结果保存到 ``destkey`` 。

- ``BITOP NOT destkey key`` ，对给定 ``key`` 求逻辑非，并将结果保存到 ``destkey`` 。

除了 ``NOT`` 操作之外，其他操作都可以接受一个或多个 ``key`` 作为输入。

**处理不同长度的字符串**

当 `BITOP`_ 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 ``0`` 。

空的 ``key`` 也被看作是包含 ``0`` 的字符串序列。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(N)

**返回值：**
    保存到 ``destkey`` 的字符串的长度，和输入 ``key`` 中最长的字符串长度相等。

.. note:: `BITOP`_ 的复杂度为 O(N) ，当处理大型矩阵(matrix)或者进行大数据量的统计时，最好将任务指派到附属节点(slave)进行，避免阻塞主节点。

::

    redis> SETBIT bits-1 0 1        # bits-1 = 1001
    (integer) 0

    redis> SETBIT bits-1 3 1
    (integer) 0

    redis> SETBIT bits-2 0 1        # bits-2 = 1011
    (integer) 0

    redis> SETBIT bits-2 1 1
    (integer) 0

    redis> SETBIT bits-2 3 1
    (integer) 0

    redis> BITOP AND and-result bits-1 bits-2
    (integer) 1

    redis> GETBIT and-result 0      # and-result = 1001
    (integer) 1

    redis> GETBIT and-result 1
    (integer) 0

    redis> GETBIT and-result 2
    (integer) 0

    redis> GETBIT and-result 3
    (integer) 1
