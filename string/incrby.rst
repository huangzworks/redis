.. _incrby:

INCRBY
======

**INCRBY key increment**

将 ``key`` 所储存的值加上增量 ``increment`` 。

如果 ``key`` 不存在，那么 ``key`` 的值会先被初始化为 ``0`` ，然后再执行 `INCRBY`_ 命令。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

关于递增(increment) / 递减(decrement)操作的更多信息，参见 :ref:`INCR` 命令。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    加上 ``increment`` 之后， ``key`` 的值。

::
    
    # key 存在且是数字值

    redis> SET rank 50 
    OK

    redis> INCRBY rank 20
    (integer) 70

    redis> GET rank  
    "70"


    # key 不存在时

    redis> EXISTS counter
    (integer) 0

    redis> INCRBY counter 30  
    (integer) 30

    redis> GET counter
    "30"


    # key 不是数字值时

    redis> SET book "long long ago..."
    OK

    redis> INCRBY book 200
    (error) ERR value is not an integer or out of range
