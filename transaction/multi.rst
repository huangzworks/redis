.. _multi:

MULTI
======

**MULTI**

标记一个事务块的开始。

事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 :ref:`EXEC` 命令原子性(atomic)地执行。

**可用版本：**
    >= 1.2.0

**时间复杂度：**
    O(1)。

**返回值：**
    总是返回 ``OK`` 。

::

    redis> MULTI            # 标记事务开始
    OK

    redis> INCR user_id     # 多条命令按顺序入队
    QUEUED

    redis> INCR user_id
    QUEUED

    redis> INCR user_id
    QUEUED

    redis> PING
    QUEUED

    redis> EXEC             # 执行
    1) (integer) 1
    2) (integer) 2
    3) (integer) 3
    4) PONG
