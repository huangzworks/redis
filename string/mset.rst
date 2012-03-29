.. _mset:

MSET
=====

**MSET key value [key value ...]**

同时设置一个或多个\ ``key-value``\ 对。

当发现同名的\ ``key``\ 存在时，\ `MSET`_\ 会用新值覆盖旧值，如果你不希望覆盖同名\ ``key``\ ，请使用\ `MSETNX`_\ 命令。  

\ `MSET`_\ 是一个原子性(atomic)操作，所有给定\ ``key``\ 都在同一时间内被设置，某些给定\ ``key``\ 被更新而另一些给定\ ``key``\ 没有改变的情况，不可能发生。

**时间复杂度：**
    O(N)，\ ``N``\ 为要设置的\ ``key``\ 数量。

**返回值：**
    总是返回\ ``OK``\ (因为\ ``MSET``\ 不可能失败)

::

    redis> MSET date "2011.4.18" time "9.09a.m." weather "sunny"    
    OK

    redis> KEYS *   # 确保指定的三个 key-value 对被插入
    1) "time"
    2) "weather"
    3) "date"

    redis> SET google "google.cn"  # MSET 覆盖旧值的例子
    OK

    redis> MSET google "google.hk"
    OK

    redis> GET google
    "google.hk"
