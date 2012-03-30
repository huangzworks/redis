.. _zadd:

ZADD
====

**ZADD key score member [[score member] [score member] ...]**

将一个或多个 ``member`` 元素及其 ``score`` 值加入到有序集 ``key`` 当中。

如果某个 ``member`` 已经是有序集的成员，那么更新这个 ``member`` 的 ``score`` 值，并通过重新插入这个 ``member`` 元素，来保证该 ``member`` 在正确的位置上。

``score`` 值可以是整数值或双精度浮点数。

如果 ``key`` 不存在，则创建一个空的有序集并执行 `ZADD`_ 操作。

当 ``key`` 存在但不是有序集类型时，返回一个错误。

对有序集的更多介绍请参见 `sorted set <http://redis.io/topics/data-types#sorted-sets>`_ 。

.. note:: 在 Redis 2.4 版本以前， `ZADD`_ 每次只能添加一个元素。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(M*log(N))， ``N`` 是有序集的基数， ``M`` 为成功添加的新成员的数量。

**返回值:**
    被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

::

    # 添加单个元素

    redis> ZADD page_rank 10 google.com
    (integer) 1


    # 添加多个元素

    redis> ZADD page_rank 9 baidu.com 8 bing.com
    (integer) 2

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"


    # 添加已存在元素，且 score 值不变

    redis> ZADD page_rank 10 google.com
    (integer) 0

    redis> ZRANGE page_rank 0 -1 WITHSCORES  # 没有改变
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"


    # 添加已存在元素，但是改变 score 值

    redis> ZADD page_rank 6 bing.com
    (integer) 0

    redis> ZRANGE page_rank 0 -1 WITHSCORES  # bing.com 元素的 score 值被改变
    1) "bing.com"
    2) "6"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"
