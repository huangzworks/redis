.. _bitcount:

BITCOUNT
===========

**BITCOUNT key [start] [end]**

计算给定字符串中，被设置为 ``1`` 的比特位的数量。

一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 ``start`` 或 ``end`` 参数，可以让计数只在特定的位上进行。

``start`` 和 ``end`` 参数的设置和 :ref:`getrange` 命令类似，都可以使用负数值：
比如 ``-1`` 表示最后一个字节， ``-2`` 表示倒数第二个字节，以此类推。

不存在的 ``key`` 被当成是空字符串来处理，因此对一个不存在的 ``key`` 进行 ``BITCOUNT`` 操作，结果为 ``0`` 。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(N)

**返回值：**
    被设置为 ``1`` 的位的数量。

::

    redis> BITCOUNT bits
    (integer) 0

    redis> SETBIT bits 0 1          # 0001
    (integer) 0

    redis> BITCOUNT bits
    (integer) 1

    redis> SETBIT bits 3 1          # 1001
    (integer) 0

    redis> BITCOUNT bits
    (integer) 2


模式：使用 bitmap 实现用户上线次数统计
-------------------------------------------

Bitmap 对于一些特定类型的计算非常有效。

假设现在我们希望记录自己网站上的用户的上线频率，比如说，计算用户 A 上线了多少天，用户 B 上线了多少天，诸如此类，以此作为数据，从而决定让哪些用户参加 beta 测试等活动 —— 这个模式可以使用 :ref:`SETBIT` 和 :ref:`BITCOUNT` 来实现。

比如说，每当用户在某一天上线的时候，我们就使用 :ref:`SETBIT` ，以用户名作为 ``key`` ，将那天所代表的网站的上线日作为 ``offset`` 参数，并将这个 ``offset`` 上的为设置为 ``1`` 。

举个例子，如果今天是网站上线的第 100 天，而用户 peter 在今天阅览过网站，那么执行命令 ``SETBIT peter 100 1`` ；如果明天 peter 也继续阅览网站，那么执行命令 ``SETBIT peter 101 1`` ，以此类推。

当要计算 peter 总共以来的上线次数时，就使用 :ref:`BITCOUNT` 命令：执行 ``BITCOUNT peter`` ，得出的结果就是 peter 上线的总天数。

更详细的实现可以参考博文(墙外) `Fast, easy, realtime metrics using Redis bitmaps <http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/>`_ 。


性能
--------

前面的上线次数统计例子，即使运行 10 年，占用的空间也只是每个用户 10*365 比特位(bit)，也即是每个用户 456 字节。对于这种大小的数据来说， :ref:`BITCOUNT` 的处理速度就像 :ref:`GET` 和 :ref:`INCR` 这种 O(1) 复杂度的操作一样快。

如果你的 bitmap 数据非常大，那么可以考虑使用以下两种方法：

1. 将一个大的 bitmap 分散到不同的 key 中，作为小的 bitmap 来处理。使用 Lua 脚本可以很方便地完成这一工作。

2. 使用 :ref:`BITCOUNT` 的 ``start`` 和 ``end`` 参数，每次只对所需的部分位进行计算，将位的累积工作(accumulating)放到客户端进行，并且对结果进行缓存 (caching)。
