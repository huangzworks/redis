.. _psetex:

PSETEX
==========

**PSETEX key milliseconds value**

这个命令和 :ref:`SETEX` 命令相似，但它以毫秒为单位设置 ``key`` 的生存时间，而不是像 :ref:`SETEX` 命令那样，以秒为单位。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(1)

**返回值：**
    设置成功时返回 ``OK`` 。

::

    redis> PSETEX mykey 1000 "Hello"
    OK

    redis> PTTL mykey
    (integer) 999

    redis> GET mykey
    "Hello"
