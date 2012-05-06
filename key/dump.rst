.. _dump:

DUMP
=========

**DUMP key**

序列化给定 ``key`` ，并返回被序列化的值，使用 :doc:`restore` 命令可以将这个值反序列化为 Redis 键。

序列化生成的值有以下几个特点：

- 它带有 64 位的校验和，用于检测错误， :doc:`restore` 在进行反序列化之前会先检查校验和。

- 值的编码格式和 RDB 文件保持一致。

- RDB 版本会被编码在序列化值当中，如果因为 Redis 的版本不同造成 RDB 格式不兼容，那么 Redis 会拒绝对这个值进行反序列化操作。

序列化的值不包括任何生存时间信息。


**可用版本：**
    >= 2.6.0

**时间复杂度：**
    | 查找给定键的复杂度为 O(1) ，对键进行序列化的复杂度为 O(N*M) ，其中 N 是构成 ``key`` 的 Redis 对象的数量，而 M 则是这些对象的平均大小。
    | 如果序列化的对象是比较小的字符串，那么复杂度为 O(1) 。

**返回值：**
    | 如果 ``key`` 不存在，那么返回 ``nil`` 。
    | 否则，返回序列化之后的值。

::

    redis> SET greeting "hello, dumping world!"
    OK

    redis> DUMP greeting
    "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"

    redis> DUMP not-exists-key
    (nil)
