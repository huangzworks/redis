.. _debug_segfault:

DEBUG SEGFAULT
===============

**DEBUG SEGFAULT**

执行一个不合法的内存访问从而让 Redis 崩溃，仅在开发时用于 BUG 模拟。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    不明确

**返回值：**
    无

::

    redis> DEBUG SEGFAULT
    Could not connect to Redis at: Connection refused

    not connected> 


