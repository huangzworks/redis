.. _brpoplpush:

BRPOPLPUSH
===========

**BRPOPLPUSH source destination timeout**

`BRPOPLPUSH`_ 是 `RPOPLPUSH`_ 的阻塞版本，当给定列表 ``source`` 不为空时， `BRPOPLPUSH`_ 的表现和 `RPOPLPUSH`_ 一样。

当列表 ``source`` 为空时， `BRPOPLPUSH`_ 命令将阻塞连接，直到等待超时，或有另一个客户端对 ``source`` 执行 `LPUSH`_ 或 `RPUSH`_ 命令为止。

超时参数 ``timeout`` 接受一个以秒为单位的数字作为值。超时参数设为 ``0`` 表示阻塞时间可以无限期延长(block indefinitely) 。

更多相关信息，请参考 `RPOPLPUSH`_ 命令。

**时间复杂度：**
    O(1)

**返回值：**
    | 假如在指定时间内没有任何元素被弹出，则返回一个\ ``nil``\ 和等待时长。
    | 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。

::

    # 情况1：非空列表

    redis> BRPOPLPUSH msg reciver 500
    "hello moto"    # 弹出元素的值
    (3.38s)         # 等待时长

    redis> LLEN reciver
    (integer) 1

    redis> LRANGE reciver 0 0
    1) "hello moto"


    # 情况2：空列表

    redis> BRPOPLPUSH msg reciver 1 
    (nil)
    (1.34s)

模式：安全队列
---------------------

参考 `RPOPLPUSH`_ 命令的『安全队列』模式。

模式：循环列表
------------------------

参考 `RPOPLPUSH`_ 命令的『循环列表』模式。
