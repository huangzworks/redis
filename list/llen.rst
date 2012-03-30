.. _llen:

LLEN
=======

**LLEN key**

返回列表 ``key`` 的长度。

如果 ``key`` 不存在，则 ``key`` 被解释为一个空列表，返回 ``0`` .

如果 ``key`` 不是列表类型，返回一个错误。 

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    列表 ``key`` 的长度。

::
    
    # 空列表

    redis> LLEN job 
    (integer) 0


    # 非空列表

    redis> LPUSH job "cook food"
    (integer) 1

    redis> LPUSH job "have lunch"
    (integer) 2

    redis> LLEN job
    (integer) 2
