.. _rpush:

RPUSH
========

**RPUSH key value [value ...]**

将一个或多个值 ``value`` 插入到列表 ``key`` 的表尾(最右边)。

如果有多个 ``value`` 值，那么各个 ``value`` 值按从左到右的顺序依次插入到表尾：比如对一个空列表 ``mylist`` 执行 ``RPUSH mylist a b c`` ，得出的结果列表为 ``a b c`` ，等同于执行命令 ``RPUSH mylist a`` 、 ``RPUSH mylist b`` 、 ``RPUSH mylist c`` 。

如果 ``key`` 不存在，一个空列表会被创建并执行 `RPUSH`_ 操作。

当 ``key`` 存在但不是列表类型时，返回一个错误。

.. note:: 在 Redis 2.4 版本以前的 `RPUSH`_ 命令，都只接受单个 ``value`` 值。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    执行 `RPUSH`_ 操作后，表的长度。

::

    # 添加单个元素

    redis> RPUSH languages c
    (integer) 1


    # 添加重复元素

    redis> RPUSH languages c
    (integer) 2

    redis> LRANGE languages 0 -1 # 列表允许重复元素
    1) "c"
    2) "c"


    # 添加多个元素

    redis> RPUSH mylist a b c
    (integer) 3

    redis> LRANGE mylist 0 -1
    1) "a"
    2) "b"
    3) "c"
