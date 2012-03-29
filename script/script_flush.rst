.. _script_flush:

SCRIPT FLUSH
==============

**SCRIPT FLUSH**

清除所有 Lua 脚本缓存。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 :ref:`EVAL` 命令。

**可用版本：**
    >= 2.6.0

**复杂度：**
    O(N) ， ``N`` 为缓存中脚本的数量。

**返回值：**
    总是返回 ``OK``

::

    redis> SCRIPT FLUSH
    OK
