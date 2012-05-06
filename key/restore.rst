.. _restore:

RESTORE
============

**RESTORE key ttl serialized-value**

反序列化给定的序列化值，并将它和给定的 ``key`` 关联。

参数 ``ttl`` 以毫秒为单位为 ``key`` 设置生存时间；如果 ``ttl`` 为 ``0`` ，那么不设置生存时间。

`RESTORE`_ 在执行反序列化之前会先对序列化值的 RDB 版本和数据校验和进行检查，如果 RDB 版本不相同或者数据不完整的话，那么 `RESTORE`_ 会拒绝进行反序列化，并返回一个错误。

更多信息可以参考 :doc:`dump` 命令。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    | 查找给定键的复杂度为 O(1) ，对键进行反序列化的复杂度为 O(N*M) ，其中 N 是构成 ``key`` 的 Redis 对象的数量，而 M 则是这些对象的平均大小。
    | 有序集合(sorted set)的反序列化复杂度为 O(N*M*log(N)) ，因为有序集合每次插入的复杂度为 O(log(N)) 。
    | 如果反序列化的对象是比较小的字符串，那么复杂度为 O(1) 。

**返回值：**
    | 如果反序列化成功那么返回 ``OK`` ，否则返回一个错误。

::

    redis> SET greeting "hello, dumping world!"
    OK

    redis> DUMP greeting
    "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"

    redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"
    OK

    redis> GET greeting-again
    "hello, dumping world!"

    redis> RESTORE fake-message 0 "hello moto moto blah blah"   ; 使用错误的值进行反序列化
    (error) ERR DUMP payload version or checksum are wrong

