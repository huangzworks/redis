.. _zcount:

ZCOUNT
=======

.. function:: ZCOUNT key min max

返回有序集\ ``key``\ 中，\ ``score``\ 值在\ ``min``\ 和\ ``max``\ 之间(默认包括\ ``score``\ 值等于\ ``min``\ 或\ ``max``\ )的成员。

关于参数\ ``min``\ 和\ ``max``\ 的详细使用方法，请参考\ `ZRANGEBYSCORE`_\ 命令。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，\ ``M``\ 为值在\ ``min``\ 和\ ``max``\ 之间的元素的数量。

**返回值:**
    \ ``score``\ 值在\ ``min``\ 和\ ``max``\ 之间的成员的数量。

::

    redis> ZRANGE salary 0 -1 WITHSCORES # 显示所有成员及其 score 值
    1) "jack"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "tom"
    6) "5000"

    redis> ZCOUNT salary 2000 5000   # 计算薪水在 2000-5000 之间的人数
    (integer) 3

    redis> ZCOUNT salary 3000 5000   # 计算薪水在 3000-5000 之间的人数
    (integer) 2


