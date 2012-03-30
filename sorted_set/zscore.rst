.. _zscore:

ZSCORE
======

**ZSCORE key member**

返回有序集 ``key`` 中，成员 ``member`` 的 ``score`` 值。

如果 ``member`` 元素不是有序集 ``key`` 的成员，或 ``key`` 不存在，返回 ``nil`` 。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(1)

**返回值:**
    ``member`` 成员的 ``score`` 值，以字符串形式表示。

::
    
    redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
    1) "tom"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "jack"
    6) "5000"

    redis> ZSCORE salary peter              # 注意返回值是字符串
    "3500"
