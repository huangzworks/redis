.. _del:

DEL
====

.. function:: DEL key [key ...]

移除给定的一个或多个\ ``key``\ 。

如果\ ``key``\ 不存在，则忽略该命令。

**时间复杂度：**
    | O(N)，\ ``N``\ 为要移除的\ ``key``\ 的数量。

    | 移除单个字符串类型的\ ``key``\ ，时间复杂度为O(1)。
    | 移除单个列表、集合、有序集合或哈希表类型的\ ``key``\ ，时间复杂度为O(M)，\ ``M``\ 为以上数据结构内的元素数量。

**返回值：**
    被移除\ ``key``\ 的数量。

::

    # 情况1： 删除单个key

    redis> SET name huangz # 设置一个值
    OK

    redis> DEL name  # 将其删除
    (integer) 1


    # 情况2： 删除一个不存在的key

    redis> EXISTS phone  # 检查一个不存在的key
    (integer) 0

    redis> DEL phone # 试图删除一个不存在的key，失败
    (integer) 0


    # 情况3： 同时删除多个key

    redis> MSET name huangz age 20 blog huangz.iteye.com # 同时设置多个key-value对
    OK

    redis> DEL name age blog # 同时删除三个key
    (integer) 3
