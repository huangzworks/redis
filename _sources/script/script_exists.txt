.. _script_exists:

SCRIPT EXISTS
=================

**SCRIPT EXISTS script [script ...]**

给定一个或多个脚本的 SHA1 校验和，返回一个包含 ``0`` 和 ``1`` 的列表，表示校验和所指定的脚本是否已经被保存在缓存当中。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 :ref:`EVAL` 命令。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(N) , ``N`` 为给定的 SHA1 校验和的数量。

**返回值：**
    | 一个列表，包含 ``0`` 和 ``1`` ，前者表示脚本不存在于缓存，后者表示脚本已经在缓存里面了。
    | 列表中的元素和给定的 SHA1 校验和保持对应关系，比如列表的第三个元素的值就表示第三个 SHA1 校验和所指定的脚本在缓存中的状态。

::

    redis> SCRIPT LOAD "return 'hello moto'"    # 载入一个脚本
    "232fd51614574cf0867b83d384a5e898cfd24e5a"

    redis> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a
    1) (integer) 1

    redis> SCRIPT FLUSH     # 清空缓存
    OK

    redis> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a
    1) (integer) 0
