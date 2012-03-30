.. _debug_object:

DEBUG OBJECT
===============

**DEBUG OBJECT key**

`DEBUG OBJECT`_ 是一个调试命令，它不应被客户端所使用。

查看 :ref:`OBJECT` 命令获取更多信息。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 当 ``key`` 存在时，返回有关信息。
    | 当 ``key`` 不存在时，返回一个错误。 

::

    redis> DEBUG OBJECT my_pc
    Value at:0xb6838d20 refcount:1 encoding:raw serializedlength:9 lru:283790 lru_seconds_idle:150

    redis> DEBUG OBJECT your_mac
    (error) ERR no such key


