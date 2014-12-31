.. _zrangebylex:

ZRANGEBYLEX
==================

**ZRANGEBYLEX key min max [LIMIT offset count]**

..
    When all the elements in a sorted set 
    are inserted with the same score,
    in order to force lexicographical ordering,
    this command returns all the elements in the sorted set at key 
    with a value between min and max.

当有序集合的所有成员都具有相同的分值时，
有序集合的元素会根据成员的字典序（lexicographical ordering）来进行排序，
而这个命令则可以返回给定的有序集合键 ``key`` 中，
值介于 ``min`` 和 ``max`` 之间的成员。

..
    If the elements in the sorted set have different scores,
    the returned elements are unspecified.

如果有序集合里面的成员带有不同的分值，
那么命令返回的结果是未指定的（unspecified）。

..
    The elements are considered to be ordered 
    from lower to higher strings 
    as compared byte-by-byte using the memcmp() C function.

    Longer strings are considered greater than shorter strings 
    if the common part is identical.

命令会使用 C 语言的 ``memcmp()`` 函数，
对集合中的每个成员进行逐个字节的对比（byte-by-byte compare），
并按照从低到高的顺序，
返回排序后的集合成员。
如果两个字符串有一部分内容是相同的话，
那么命令会认为较长的字符串比较短的字符串要大。

..
    The optional LIMIT argument 
    can be used to only get a range of the matching elements 
    (similar to SELECT LIMIT offset, count in SQL).

    Keep in mind that if offset is large,
    the sorted set needs to be traversed for offset elements 
    before getting to the elements to return,
    which can add up to O(N) time complexity.

可选的 ``LIMIT offset count`` 参数用于获取指定范围内的匹配元素
（就像 SQL 中的 ``SELECT LIMIT offset count`` 语句）。
需要注意的一点是，
如果 ``offset`` 参数的值非常大的话，
那么命令在返回结果之前，
需要先遍历至 ``offset`` 所指定的位置，
这个操作会为命令加上最多 O(N) 复杂度。

..
    How to specify intervals
    ---------------------------

    Valid start and stop must start with ( or [ ,
    in order to specify if the range item is respectively exclusive or inclusive. 

    The special values of + or - for start and stop 
    have the special meaning of positively infinite and negatively infinite strings,
    so for instance the command ``ZRANGEBYLEX myzset - +`` 
    is guaranteed to return all the elements in the sorted set,
    if all the elements have the same score.

如何指定范围区间
-----------------------

合法的 ``min`` 和 ``max`` 参数必须包含 ``(`` 或者 ``[`` ，
其中 ``(`` 表示开区间（指定的值不会被包含在范围之内），
而 ``[`` 则表示闭区间（指定的值会被包含在范围之内）。

特殊值 ``+`` 和 ``-`` 在 ``min`` 参数以及 ``max`` 参数中具有特殊的意义，
其中 ``+`` 表示正无限，
而 ``-`` 表示负无限。
因此，
向一个所有成员的分值都相同的有序集合发送命令 ``ZRANGEBYLEX <zset> - +`` ，
命令将返回有序集合中的所有元素。

..
    Details on strings comparison
    --------------------------------

    关于字符串对比的细节
    ------------------------

..
    Strings are compared as binary array of bytes.
    Because of how the ASCII character set is specified, 
    this means that usually this also have the effect of comparing normal ASCII characters in an obvious dictionary way. 
    However this is not true if non plain ASCII strings are used 
    (for example utf8 strings).

..
    Redis 会将字符串看作是二进制字节数组（binary array of bytes）来进行对比，
    在 ASCII 字符集上，
    这种对比方式和普通的 ASCII 对比方式都可以产生字典序排列结果，
    但对于非 ASCII 字符集（比如 utf8 字符集来说），
    普通字符串排序就没办法产生字典序排列结果了。

..
    However the user can apply a transformation to the encoded string 
    so that the first part of the element inserted in the sorted set 
    will compare as the user requires for the specific application.
    For example 
    if I want to add strings that will be compared in a case-insensitive way,
    but I still want to retrieve the real case when querying, 
    I can add strings in the following way:

    ::

        ZADD autocomplete 0 foo:Foo 0 bar:BAR 0 zap:zap

..
    Because of the first normalized part in every element (before the colon character),
    we are forcing a given comparison, 
    however after the range is queries using ZRANGEBYLEX 
    the application can display to the user 
    the second part of the string,
    after the colon.

..
    The binary nature of the comparison allows to use sorted sets as a general purpose index, 
    for example the first part of the element can be a 64 bit big endian number: 
    since big endian numbers have the most significant bytes in the initial positions, 
    the binary comparison will match the numerical comparison of the numbers. 
    This can be used in order to implement range queries on 64 bit values. 
    As in the example below, 
    after the first 8 bytes we can store the value of the element we are actually indexing.


**可用版本：**
    >= 2.8.9


**时间复杂度：**
    O(log(N)+M)，
    其中 N 为有序集合的元素数量，
    而 M 则是命令返回的元素数量。
    如果 M 是一个常数（比如说，用户总是使用 ``LIMIT`` 参数来返回最先的 10 个元素），
    那么命令的复杂度也可以看作是 O(log(N)) 。

..
    O(log(N)+M) with N being the number of elements in the sorted set 
    and M the number of elements being returned. 
    If M is constant (e.g. always asking for the first 10 elements with LIMIT), 
    you can consider it O(log(N)).


**返回值：**
    数组回复：一个列表，列表里面包含了有序集合在指定范围内的成员。

..  Array reply: list of elements in the specified score range.


::

    redis> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
    (integer) 7

    redis> ZRANGEBYLEX myzset - [c
    1) "a"
    2) "b"
    3) "c"

    redis> ZRANGEBYLEX myzset - (c
    1) "a"
    2) "b"

    redis> ZRANGEBYLEX myzset [aaa (g
    1) "b"
    2) "c"
    3) "d"
    4) "e"
    5) "f"
