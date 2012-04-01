.. _del:

DEL
====

**DEL key [key ...]**

删除给定的一个或多个 ``key`` 。

不存在的 ``key`` 会被忽略。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    | O(N)， ``N`` 为被删除的 ``key`` 的数量。

    | 删除单个字符串类型的 ``key`` ，时间复杂度为O(1)。
    | 删除单个列表、集合、有序集合或哈希表类型的 ``key`` ，时间复杂度为O(M)， ``M`` 为以上数据结构内的元素数量。

**返回值：**
    被删除 ``key`` 的数量。

::

    #  删除单个 key

    redis> SET name huangz 
    OK

    redis> DEL name  
    (integer) 1


    # 删除一个不存在的 key

    redis> EXISTS phone 
    (integer) 0

    redis> DEL phone # 失败，没有 key 被删除
    (integer) 0


    # 同时删除多个 key

    redis> SET name "redis"
    OK

    redis> SET type "key-value store"
    OK

    redis> SET website "redis.com"
    OK

    redis> DEL name type website
    (integer) 3
