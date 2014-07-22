.. _zlexcount:

ZLEXCOUNT
==============

**ZLEXCOUNT key min max**

When all the elements in a sorted set are inserted with the same score, in order to force lexicographical ordering, this command returns the number of elements in the sorted set at key with a value between min and max.

The min and max arguments have the same meaning as described for ZRANGEBYLEX.

Note: the command has a complexity of just O(log(N)) because it uses elements ranks (see ZRANK) to get an idea of the range. Because of this there is no need to do a work proportional to the size of the range.

**可用版本：**
    >= 2.8.9

**时间复杂度：**
    O(log(N)) with N being the number of elements in the sorted set.

**返回值：**
    Integer reply: the number of elements in the specified score range.

::

    redis> ZADD myzset 0 a 0 b 0 c 0 d 0 e
    (integer) 5

    redis> ZADD myzset 0 f 0 g
    (integer) 2

    redis> ZLEXCOUNT myzset - +
    (integer) 7

    redis> ZLEXCOUNT myzset [b [f
    (integer) 5
