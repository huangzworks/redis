.. _decr:

DECR
=====

**DECR key**

将 ``key`` 中储存的数字值减一。

如果 ``key`` 不存在，那么 ``key`` 的值会先被初始化为 ``0`` ，然后再执行 `DECR`_ 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

关于递增(increment) / 递减(decrement)操作的更多信息，请参见 :ref:`INCR` 命令。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    执行 `DECR`_ 命令之后 ``key`` 的值。

::

    # 对存在的数字值 key 进行 DECR

    redis> SET failure_times 10
    OK

    redis> DECR failure_times
    (integer) 9


    # 对不存在的 key 值进行 DECR

    redis> EXISTS count 
    (integer) 0

    redis> DECR count
    (integer) -1


    # 对存在但不是数值的 key 进行 DECR

    redis> SET company YOUR_CODE_SUCKS.LLC
    OK

    redis> DECR company
    (error) ERR value is not an integer or out of range
