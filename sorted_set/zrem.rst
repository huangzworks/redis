.. _zrem:

ZREM
=====

**ZREM key member [member ...]**

移除有序集 ``key`` 中的一个或多个成员，不存在的成员将被忽略。

当 ``key`` 存在但不是有序集类型时，返回一个错误。

.. note:: 在 Redis 2.4 版本以前， `ZREM`_ 每次只能删除一个元素。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(M*log(N))， ``N`` 为有序集的基数， ``M`` 为被成功移除的成员的数量。

**返回值:**
    被成功移除的成员的数量，不包括被忽略的成员。


::

    # 测试数据

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"


    # 移除单个元素

    redis> ZREM page_rank google.com
    (integer) 1

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"


    # 移除多个元素

    redis> ZREM page_rank baidu.com bing.com
    (integer) 2

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    (empty list or set)


    # 移除不存在元素

    redis> ZREM page_rank non-exists-element
    (integer) 0
