.. _setbit:

SETBIT
=======

**SETBIT key offset value**

对 ``key`` 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

位的设置或清除取决于 ``value`` 参数，可以是 ``0`` 也可以是 ``1`` 。

当 ``key`` 不存在时，自动生成一个新的字符串值。

字符串会进行伸展(grown)以确保它可以将 ``value`` 保存在指定的偏移量上。当字符串值进行伸展时，空白位置以 ``0`` 填充。

``offset`` 参数必须大于或等于 ``0`` ，小于 2^32 (bit 映射被限制在 512 MB 之内)。

.. warning:: 对使用大的 ``offset`` 的 `SETBIT`_ 操作来说，内存分配可能造成 Redis 服务器被阻塞。具体参考 :ref:`SETRANGE` 命令，warning(警告)部分。

**可用版本：**
    >= 2.2.0

**时间复杂度:**
    O(1)

**返回值：**
    指定偏移量原来储存的位。

::

    redis> SETBIT bit 10086 1
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 1

    redis> GETBIT bit 100   # bit 默认被初始化为 0 
    (integer) 0
