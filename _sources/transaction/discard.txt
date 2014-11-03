.. _discard:

DISCARD
=========

**DISCARD**

取消事务，放弃执行事务块内的所有命令。

如果正在使用 :ref:`WATCH` 命令监视某个(或某些) key，那么取消所有监视，等同于执行命令 :ref:`UNWATCH` 。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)。

**返回值：**
    总是返回 ``OK`` 。

::

    redis> MULTI
    OK

    redis> PING
    QUEUED

    redis> SET greeting "hello"
    QUEUED

    redis> DISCARD
    OK
