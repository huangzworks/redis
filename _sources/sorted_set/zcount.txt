.. _zcount:

ZCOUNT
=======

**ZCOUNT key min max**

返回有序集 ``key`` 中， ``score`` 值在 ``min`` 和 ``max`` 之间(默认包括 ``score`` 值等于 ``min`` 或 ``max`` )的成员的数量。

关于参数 ``min`` 和 ``max`` 的详细使用方法，请参考 :ref:`ZRANGEBYSCORE` 命令。

**可用版本：**
    >= 2.0.0

**时间复杂度:**
    O(log(N))， ``N`` 为有序集的基数。

**返回值:**
     ``score`` 值在 ``min`` 和 ``max`` 之间的成员的数量。

::

    redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
    1) "jack"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "tom"
    6) "5000"

    redis> ZCOUNT salary 2000 5000          # 计算薪水在 2000-5000 之间的人数
    (integer) 3

    redis> ZCOUNT salary 3000 5000          # 计算薪水在 3000-5000 之间的人数
    (integer) 2
