.. _llen:

LLEN
=======

.. function:: LLEN key

返回列表\ ``key``\ 的长度。

如果\ ``key``\ 不存在，则\ ``key``\ 被解释为一个空列表，返回\ ``0``\ .

如果\ ``key``\ 不是列表类型，返回一个错误。 

**时间复杂度：**
    O(1)

**返回值：**
    列表\ ``key``\ 的长度。

::
    
    # 情况1：空列表

    redis> LLEN job 
    (integer) 0


    # 情况2：非空列表

    redis> LPUSH job "cook food"
    (integer) 1
    redis> LPUSH job "have lunch"
    (integer) 2

    redis> LLEN job
    (integer) 2


