.. _getbit:

GETBIT
======

**GETBIT key offset**

对\ ``key``\ 所储存的字符串值，获取指定偏移量上的位(bit)。

当\ ``offset``\ 比字符串值的长度大，或者\ ``key``\ 不存在时，返回\ ``0``\ 。
            
**时间复杂度：**
    O(1)

**返回值：**
    字符串值指定偏移量上的位(bit)。

::
    
    # 情况1：对不存在的 key 或不存在的 offset 进行 GETBIT，
    #        默认为0

    redis> EXISTS bit
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 0

    
    # 情况2：对已存在的 offset 进行 GETBIT

    redis> SETBIT bit 10086 1
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 1
