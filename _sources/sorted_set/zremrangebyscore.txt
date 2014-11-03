.. _zremrangebyscore:

ZREMRANGEBYSCORE
=================

**ZREMRANGEBYSCORE key min max**

移除有序集 ``key`` 中，所有 ``score`` 值介于 ``min`` 和 ``max`` 之间(包括等于 ``min`` 或 ``max`` )的成员。

自版本2.1.6开始， ``score`` 值等于 ``min`` 或 ``max`` 的成员也可以不包括在内，详情请参见 :ref:`ZRANGEBYSCORE` 命令。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(log(N)+M)， ``N`` 为有序集的基数，而 ``M`` 为被移除成员的数量。

**返回值:**
    被移除成员的数量。

::
    
    redis> ZRANGE salary 0 -1 WITHSCORES          # 显示有序集内所有成员及其 score 值
    1) "tom"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "jack"
    6) "5000"

    redis> ZREMRANGEBYSCORE salary 1500 3500      # 移除所有薪水在 1500 到 3500 内的员工
    (integer) 2

    redis> ZRANGE salary 0 -1 WITHSCORES          # 剩下的有序集成员
    1) "jack"
    2) "5000"



