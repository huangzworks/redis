.. _decrby:

DECRBY
=======

**DECRBY key decrement**

将 ``key`` 所储存的值减去减量 ``decrement`` 。

如果 ``key`` 不存在，那么 ``key`` 的值会先被初始化为 ``0`` ，然后再执行 `DECRBY`_ 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

关于更多递增(increment) / 递减(decrement)操作的更多信息，请参见 :ref:`INCR` 命令。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    减去 ``decrement`` 之后， ``key`` 的值。

::

    # 对已存在的 key 进行 DECRBY

    redis> SET count 100
    OK

    redis> DECRBY count 20
    (integer) 80

    
    # 对不存在的 key 进行DECRBY

    redis> EXISTS pages 
    (integer) 0

    redis> DECRBY pages 10  
    (integer) -10
