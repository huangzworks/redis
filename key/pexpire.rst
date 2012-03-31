.. _pexpire:

PEXPIRE
========

**PEXPIRE key milliseconds**

这个命令和 :doc:`expire` 命令的作用类似，但是它以毫秒为单位设置 ``key`` 的生存时间，而不像 :doc:`expire` 命令那样，以秒为单位。

**可用版本：**
    >= 2.6.0

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

    redis> TTL mykey    # TTL 的返回值以秒为单位
    (integer) 2

    redis> PTTL mykey   # PTTL 可以给出准确的毫秒数
    (integer) 1499
