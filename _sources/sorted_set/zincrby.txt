.. _zincrby:

ZINCRBY
========

**ZINCRBY key increment member**

为有序集 ``key`` 的成员 ``member`` 的 ``score`` 值加上增量 ``increment`` 。

可以通过传递一个负数值 ``increment`` ，让 ``score`` 减去相应的值，比如 ``ZINCRBY key -5 member`` ，就是让 ``member`` 的 ``score`` 值减去 ``5`` 。

当 ``key`` 不存在，或 ``member`` 不是 ``key`` 的成员时， ``ZINCRBY key increment member`` 等同于 ``ZADD key increment member`` 。

当 ``key`` 不是有序集类型时，返回一个错误。

``score`` 值可以是整数值或双精度浮点数。

**可用版本：**
    >= 1.2.0

**时间复杂度:**
    O(log(N))

**返回值:**
    ``member`` 成员的新 ``score`` 值，以字符串形式表示。

::

    redis> ZSCORE salary tom 
    "2000"

    redis> ZINCRBY salary 2000 tom   # tom 加薪啦！
    "4000"
