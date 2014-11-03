.. _exec:

EXEC
======

**EXEC**

执行所有事务块内的命令。

假如某个(或某些) key 正处于 :ref:`WATCH` 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 `EXEC`_ 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。

**可用版本：**
    >= 1.2.0

**时间复杂度：**
    事务块内所有命令的时间复杂度的总和。

**返回值：**
    | 事务块内所有命令的返回值，按命令执行的先后顺序排列。
    | 当操作被打断时，返回空值 ``nil`` 。

::

    # 事务被成功执行

    redis> MULTI
    OK

    redis> INCR user_id
    QUEUED

    redis> INCR user_id
    QUEUED

    redis> INCR user_id
    QUEUED

    redis> PING
    QUEUED

    redis> EXEC
    1) (integer) 1
    2) (integer) 2
    3) (integer) 3
    4) PONG


    # 监视 key ，且事务成功执行

    redis> WATCH lock lock_times
    OK

    redis> MULTI
    OK

    redis> SET lock "huangz"
    QUEUED

    redis> INCR lock_times
    QUEUED

    redis> EXEC
    1) OK
    2) (integer) 1


    # 监视 key ，且事务被打断 

    redis> WATCH lock lock_times
    OK

    redis> MULTI
    OK

    redis> SET lock "joe"        # 就在这时，另一个客户端修改了 lock_times 的值
    QUEUED

    redis> INCR lock_times
    QUEUED

    redis> EXEC                  # 因为 lock_times 被修改， joe 的事务执行失败
    (nil)
