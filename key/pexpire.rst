.. _pexipre:

PEXPIRE
========

.. function:: PEXPIRE key milliseconds

这个命令和 :ref:`EXPIRE` 命令的作用类似，但是它以毫秒为单位设置 ``key`` 的生存时间，而不像 :ref:`EXPIRE` 命令那样，以秒为单位。

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功，返回 ``1`` 
    | ``key`` 不存在或设置失败，返回 ``0`` 

::

    redis> SET mykey "Hello"
    OK

    redis> PEXPIRE mykey 1500
    (integer) 1

    redis> TTL mykey
    (integer) 2

    redis> PTTL mykey
    (integer) 1499
 
