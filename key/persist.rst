.. _persist:

PERSIST
========

.. function:: PERSIST key

移除给定 ``key`` 的生存时间，将这个 ``key`` 从『可挥发』的(带生存时间 ``key`` )转换成『持久化』的(一个不带生存时间、永不过期的 ``key`` )。

**时间复杂度：**
    O(1)

**返回值：**
    | 当生存时间移除成功时，返回\ ``1``\ .
    | 如果\ ``key``\ 不存在或\ ``key``\ 没有设置生存时间，返回\ ``0``\ 。

::

    redis> SET time_to_say_goodbye "886..."
    OK

    redis> EXPIRE time_to_say_goodbye 300
    (integer) 1

    redis> TTL time_to_say_goodbye
    (integer) 293
    
    redis> PERSIST time_to_say_goodbye  # 移除生存时间
    (integer) 1
    
    redis> TTL time_to_say_goodbye  # 移除成功
    (integer) -1



