.. _hmset:

HMSET
=====

**HMSET key field value [field value ...]**

同时将多个 ``field-value`` (域-值)对设置到哈希表 ``key`` 中。

此命令会覆盖哈希表中已存在的域。

如果 ``key`` 不存在，一个空哈希表被创建并执行 `HMSET`_ 操作。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(N)， ``N`` 为 ``field-value`` 对的数量。

**返回值：**
    | 如果命令执行成功，返回 ``OK`` 。
    | 当 ``key`` 不是哈希表(hash)类型时，返回一个错误。

::

    redis> HMSET website google www.google.com yahoo www.yahoo.com 
    OK

    redis> HGET website google
    "www.google.com"

    redis> HGET website yahoo
    "www.yahoo.com"
