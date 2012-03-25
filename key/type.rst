.. _type:

TYPE
=====

.. function:: TYPE key

返回\ ``key``\ 所储存的值的类型。

**时间复杂度：**
    O(1)

**返回值：**
    | \ ``none``\ (key不存在)
    | \ ``string``\ (字符串)
    | \ ``list``\ (列表)
    | \ ``set``\ (集合)
    | \ ``zset``\ (有序集)
    | \ ``hash``\ (哈希表)

::

    redis> SET weather "sunny"  # 构建一个字符串
    OK

    redis> TYPE weather 
    string

    redis> LPUSH book_list "programming in scala"  # 构建一个列表
    (integer) 1

    redis> TYPE book_list 
    list

    redis> SADD pat "dog"  # 构建一个集合
    (integer) 1

    redis> TYPE pat
    set


