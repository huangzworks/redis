.. _brpop:

BRPOP
=======

**BRPOP key [key ...] timeout**

`BRPOP`_ 是列表的阻塞式(blocking)弹出原语。

它是 :ref:`RPOP` 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 `BRPOP`_ 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 ``key`` 参数时，按参数 ``key`` 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。

关于阻塞操作的更多信息，请查看 :ref:`BLPOP` 命令， `BRPOP`_ 除了弹出元素的位置和 :ref:`BLPOP` 不同之外，其他表现一致。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 假如在指定时间内没有任何元素被弹出，则返回一个 ``nil`` 和等待时长。
    | 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 ``key`` ，第二个元素是被弹出元素的值。

::

    redis> LLEN course
    (integer) 0

    redis> RPUSH course algorithm001
    (integer) 1

    redis> RPUSH course c++101
    (integer) 2

    redis> BRPOP course 30
    1) "course"             # 被弹出元素所属的列表键
    2) "c++101"             # 被弹出的元素
