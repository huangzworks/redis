.. _echo:

ECHO
=======

**ECHO message**

打印一个特定的信息 ``message`` ，测试时使用。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    ``message`` 自身。

::

    redis> ECHO "Hello Moto"
    "Hello Moto"

    redis> ECHO "Goodbye Moto"
    "Goodbye Moto"
