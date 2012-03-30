.. _watch:

WATCH
======

**WATCH key [key ...]**

监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

**可用版本：**
    >= 2.2.0

**时间复杂度：**
    O(1)。

**返回值：**
    总是返回 ``OK`` 。

::

    redis> WATCH lock lock_times
    OK
