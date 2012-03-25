.. _decr:

DECR
=====

.. function:: DECR key

将\ ``key``\ 中储存的数字值减一。

如果\ ``key``\ 不存在，以\ ``0``\ 为\ ``key``\ 的初始值，然后执行\ `DECR`_\ 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在64位(bit)有符号数字表示之内。

关于更多递增(increment)/递减(decrement)操作信息，参见\ `INCR`_\ 命令。

**时间复杂度：**
    O(1)

**返回值：**
    执行\ `DECR`_\ 命令之后\ ``key``\ 的值。

::

    # 情况1：对存在的数字值key进行DECR

    redis> SET failure_times 10
    OK

    redis> DECR failure_times
    (integer) 9


    # 情况2：对不存在的key值进行DECR

    redis> EXISTS count 
    (integer) 0

    redis> DECR count
    (integer) -1


    # 情况3：对存在但不是数值的key进行DECR

    redis> SET company YOUR_CODE_SUCKS.LLC
    OK

    redis> DECR company
    (error) ERR value is not an integer or out of range


