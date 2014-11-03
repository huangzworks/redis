.. _lrange:

LRANGE
=======

**LRANGE key start stop**

返回列表 ``key`` 中指定区间内的元素，区间以偏移量 ``start`` 和 ``stop`` 指定。

下标(index)参数 ``start`` 和 ``stop`` 都以 ``0`` 为底，也就是说，以 ``0`` 表示列表的第一个元素，以 ``1`` 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以 ``-1`` 表示列表的最后一个元素， ``-2`` 表示列表的倒数第二个元素，以此类推。

**注意LRANGE命令和编程语言区间函数的区别**

假如你有一个包含一百个元素的列表，对该列表执行 ``LRANGE list 0 10`` ，结果是一个包含11个元素的列表，这表明 ``stop`` 下标也在 `LRANGE`_ 命令的取值范围之内(闭区间)，这和某些语言的区间函数可能不一致，比如Ruby的 ``Range.new`` 、 ``Array#slice`` 和Python的 ``range()`` 函数。

**超出范围的下标**

超出范围的下标值不会引起错误。

如果 ``start`` 下标比列表的最大下标 ``end`` ( ``LLEN list`` 减去 ``1`` )还要大，那么 `LRANGE`_ 返回一个空列表。

如果 ``stop`` 下标比 ``end`` 下标还要大，Redis将 ``stop`` 的值设置为 ``end`` 。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(S+N)， ``S`` 为偏移量 ``start`` ， ``N`` 为指定区间内元素的数量。

**返回值:**
    一个列表，包含指定区间内的元素。

::

    redis> RPUSH fp-language lisp   
    (integer) 1

    redis> LRANGE fp-language 0 0 
    1) "lisp"

    redis> RPUSH fp-language scheme
    (integer) 2

    redis> LRANGE fp-language 0 1
    1) "lisp"
    2) "scheme"



