.. _zlexcount:

ZLEXCOUNT
==============

**ZLEXCOUNT key min max**

..
    When all the elements in a sorted set are inserted with the same score,
    in order to force lexicographical ordering,
    this command returns the number of elements in the sorted set 
    at key with a value between min and max.

对于一个所有成员的分值都相同的有序集合键 ``key`` 来说，
这个命令会返回该集合中，
成员介于 ``min`` 和 ``max`` 范围内的元素数量。

..  
    The min and max arguments have the same meaning as described for :ref:`ZRANGEBYLEX` .

这个命令的 ``min`` 参数和 ``max`` 参数的意义和 :ref:`ZRANGEBYLEX` 命令的 ``min`` 参数和 ``max`` 参数的意义一样。

..
    Note: the command has a complexity of just O(log(N)) 
          because it uses elements ranks (see ZRANK) to get an idea of the range.
          Because of this there is no need to do a work proportional to the size of the range.


**可用版本：**
    >= 2.8.9


**时间复杂度：**
    O(log(N))，其中 N 为有序集合包含的元素数量。

..  O(log(N)) with N being the number of elements in the sorted set.


**返回值：**
    整数回复：指定范围内的元素数量。

..  Integer reply: the number of elements in the specified score range.

::

    redis> ZADD myzset 0 a 0 b 0 c 0 d 0 e
    (integer) 5

    redis> ZADD myzset 0 f 0 g
    (integer) 2

    redis> ZLEXCOUNT myzset - +
    (integer) 7

    redis> ZLEXCOUNT myzset [b [f
    (integer) 5
