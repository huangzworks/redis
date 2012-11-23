.. _setrange:

SETRANGE
=========

**SETRANGE key offset value**

用 ``value`` 参数覆写(overwrite)给定 ``key`` 所储存的字符串值，从偏移量 ``offset`` 开始。

不存在的 ``key`` 当作空白字符串处理。

`SETRANGE`_ 命令会确保字符串足够长以便将 ``value`` 设置在指定的偏移量上，如果给定 ``key`` 原来储存的字符串长度比偏移量小(比如字符串只有 ``5`` 个字符长，但你设置的 ``offset`` 是 ``10`` )，那么原字符和偏移量之间的空白将用零字节(zerobytes, ``"\x00"`` )来填充。

注意你能使用的最大偏移量是 2^29-1(536870911) ，因为 Redis 字符串的大小被限制在 512 兆(megabytes)以内。如果你需要使用比这更大的空间，你可以使用多个 ``key`` 。

.. warning:: 
    当生成一个很长的字符串时，Redis 需要分配内存空间，该操作有时候可能会造成服务器阻塞(block)。在2010年的Macbook Pro上，设置偏移量为 536870911(512MB 内存分配)，耗费约 300 毫秒，
    设置偏移量为 134217728(128MB 内存分配)，耗费约 80 毫秒，设置偏移量 33554432(32MB 内存分配)，耗费约 30 毫秒，设置偏移量为 8388608(8MB 内存分配)，耗费约 8 毫秒。
    注意若首次内存分配成功之后，再对同一个 ``key`` 调用 `SETRANGE`_ 操作，无须再重新内存。

**可用版本：**
    >= 2.2.0

**时间复杂度：**
    | 对小(small)的字符串，平摊复杂度O(1)。(关于什么字符串是"小"的，请参考 :ref:`APPEND` 命令)
    | 否则为O(M)， ``M`` 为 ``value`` 参数的长度。

**返回值：**
    被 `SETRANGE`_ 修改之后，字符串的长度。

::

    # 对非空字符串进行 SETRANGE

    redis> SET greeting "hello world" 
    OK

    redis> SETRANGE greeting 6 "Redis"
    (integer) 11

    redis> GET greeting
    "hello Redis"


    # 对空字符串/不存在的 key 进行 SETRANGE

    redis> EXISTS empty_string
    (integer) 0

    redis> SETRANGE empty_string 5 "Redis!"   # 对不存在的 key 使用 SETRANGE
    (integer) 11

    redis> GET empty_string                   # 空白处被"\x00"填充
    "\x00\x00\x00\x00\x00Redis!"

模式
-------

因为有了 `SETRANGE`_ 和 :ref:`GETRANGE` 命令，你可以将 Redis 字符串用作具有O(1)随机访问时间的线性数组，这在很多真实用例中都是非常快速且高效的储存方式，具体请参考 :ref:`APPEND` 命令的『模式：时间序列』部分。
