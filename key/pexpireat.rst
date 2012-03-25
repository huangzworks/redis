.. _pexpireat:

PEXPIREAT
============

.. function:: key milliseconds timestamp

这个命令和 :ref:`EXPIREAT` 命令类似，但它以毫秒为单位设置 ``key`` 的过期 UNIX 时间戳，而不是像 :ref:`EXPIREAT` 那样，以秒为单位。

**时间复杂度：**
    O(1)

**返回值：**
    | 如果生存时间设置成功，返回 ``1`` 。
    | 当 ``key`` 不存在或没办法设置生存时间时，返回 ``0`` 。(查看 :ref:`EXPIRE` 命令获取更多信息)

::

    redis> SET mykey "Hello"
    OK

    redis> PEXPIREAT mykey 1555555555005
    (integer) 1

    redis> TTL mykey
    (integer) 223157079

    redis> PTTL mykey
    (integer) 223157079318


