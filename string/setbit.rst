.. _setbit:

SETBIT
=======

.. function:: SETBIT key offset value 

对\ ``key``\ 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

位的设置或清除取决于\ ``value``\ 参数，可以是\ ``0``\ 也可以是\ ``1``\ 。

当\ ``key``\ 不存在时，自动生成一个新的字符串值。

字符串会增长(grown)以确保它可以将\ ``value``\ 保存在指定的偏移量上。当字符串值增长时，空白位置以\ ``0``\ 填充。

\ ``offset``\ 参数必须大于或等于\ ``0``\ ，小于2^32(bit映射被限制在512MB内)。


**时间复杂度:**
    O(1)

**返回值：**
    指定偏移量原来储存的位。

.. warning:: 对使用大的\ ``offset``\ 的\ `SETBIT`_\ 操作来说，内存分配可能造成 Redis 服务器被阻塞。具体参考\ `SETRANGE`_\ 命令，warning(警告)部分。

::

    redis> SETBIT bit 10086 1
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 1


