.. _lpush:

LPUSH
=======

**LPUSH key value [value ...]**

将一个或多个值 ``value`` 插入到列表 ``key`` 的表头

如果有多个 ``value`` 值，那么各个 ``value`` 值按从左到右的顺序依次插入到表头：
比如说，对空列表 ``mylist`` 执行命令 ``LPUSH mylist a b c`` ，列表的值将是 ``c b a`` ，这等同于原子性地执行 ``LPUSH mylist a`` 、 ``LPUSH mylist b`` 和 ``LPUSH mylist c`` 三个命令。

如果 ``key`` 不存在，一个空列表会被创建并执行 `LPUSH`_ 操作。

当 ``key`` 存在但不是列表类型时，返回一个错误。

.. note:: 在Redis 2.4版本以前的 `LPUSH`_ 命令，都只接受单个 ``value`` 值。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    执行 `LPUSH`_ 命令后，列表的长度。

::
    
    # 加入单个元素

    redis> LPUSH languages python
    (integer) 1


    # 加入重复元素

    redis> LPUSH languages python
    (integer) 2

    redis> LRANGE languages 0 -1     # 列表允许重复元素
    1) "python"
    2) "python"


    # 加入多个元素

    redis> LPUSH mylist a b c
    (integer) 3

    redis> LRANGE mylist 0 -1
    1) "c"
    2) "b"
    3) "a"
