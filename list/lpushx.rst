.. _lpushx:

LPUSHX
=======

.. function:: LPUSHX key value

将值\ ``value``\ 插入到列表\ ``key``\ 的表头，当且仅当\ ``key``\ 存在并且是一个列表。

和\ `LPUSH`_\ 命令相反，当\ ``key``\ 不存在时，\ `LPUSHX`_\ 命令什么也不做。
            
**时间复杂度：**
    O(1)

**返回值：**
    \ `LPUSHX`_\ 命令执行之后，表的长度。

::

    # 情况1：对空列表执行LPUSHX

    redis> LLEN greet    # greet是一个空列表
    (integer) 0

    redis> LPUSHX greet "hello"  # 尝试LPUSHX，失败，因为列表为空
    (integer) 0

    
    # 情况2：对非空列表执行LPUSHX

    redis> LPUSH greet "hello"   # 先用LPUSH创建一个有一个元素的列表
    (integer) 1

    redis> LPUSHX greet "good morning"   # 这次LPUSHX执行成功
    (integer) 2

    redis> LRANGE greet 0 -1
    1) "good morning"
    2) "hello"



