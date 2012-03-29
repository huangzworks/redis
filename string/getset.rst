.. _getset:

GETSET
========

**GETSET key value**

将给定\ ``key``\ 的值设为\ ``value``\ ，并返回\ ``key``\ 的旧值。

当\ ``key``\ 存在但不是字符串类型时，返回一个错误。

**时间复杂度：**
    O(1)

**返回值：**
    | 返回给定\ ``key``\ 的旧值(old value)。
    | 当\ ``key``\ 没有旧值时，返回\ ``nil``\ 。

::

    redis> EXISTS mail 
    (integer) 0

    redis> GETSET mail xxx@google.com  # 因为 mail 之前不存在，没有旧值，返回 nil
    (nil)

    redis> GETSET mail xxx@yahoo.com  # mail 被更新，旧值被返回
    "xxx@google.com"

模式
--------

\ `GETSET`_\ 可以和\ `INCR`_\ 组合使用，实现一个有原子性(atomic)复位操作的计数器(counter)。

举例来说，每次当某个事件发生时，进程可能对一个名为\ ``mycount``\ 的\ ``key``\ 调用\ `INCR`_\ 操作，通常我们还要在一个原子时间内同时完成获得计数器的值和将计数器值复位为\ ``0``\ 两个操作。

可以用命令\ ``GETSET mycounter 0``\ 来实现这一目标。

::
    
    redis> INCR mycount 
    (integer) 11

    redis> GETSET mycount 0  # 一个原子内完成 GET mycount 和 SET mycount 0 操作
    "11"

    redis> GET mycount
    "0"


