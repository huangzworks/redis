.. _mset:

MSET
=====

**MSET key value [key value ...]**

同时设置一个或多个 ``key-value`` 对。

如果某个给定 ``key`` 已经存在，那么 `MSET`_ 会用新值覆盖原来的旧值，如果这不是你所希望的效果，请考虑使用 :ref:`MSETNX` 命令：它只会在所有给定 ``key`` 都不存在的情况下进行设置操作。

`MSET`_ 是一个原子性(atomic)操作，所有给定 ``key`` 都会在同一时间内被设置，某些给定 ``key`` 被更新而另一些给定 ``key`` 没有改变的情况，不可能发生。

**可用版本：**
    >= 1.0.1

**时间复杂度：**
    O(N)， ``N`` 为要设置的 ``key`` 数量。

**返回值：**
    总是返回 ``OK`` (因为 ``MSET`` 不可能失败)

::

    redis> MSET date "2012.3.30" time "11:00 a.m." weather "sunny"
    OK

    redis> MGET date time weather   
    1) "2012.3.30"
    2) "11:00 a.m."
    3) "sunny"


    # MSET 覆盖旧值例子

    redis> SET google "google.hk"       
    OK

    redis> MSET google "google.com"
    OK

    redis> GET google
    "google.com"
