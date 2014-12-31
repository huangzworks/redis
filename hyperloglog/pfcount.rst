.. _pfcount:

PFCOUNT
=============

**PFCOUNT key [key ...]**

..
    When call with a single key, 
    returns the approximated cardinality 
    computed by the HyperLogLog data structure stored at the specified variable, 
    which is 0 if the variable does not exist.

当 :ref:`PFCOUNT` 命令作用于单个键时，
返回储存在给定键的 HyperLogLog 的近似基数，
如果键不存在，
那么返回 ``0`` 。

..
    When called with multiple keys, 
    returns the approximated cardinality of the union of the HyperLogLogs passed, 
    by internally merging the HyperLogLogs stored at the provided keys into a temporary hyperLogLog.

当 :ref:`PFCOUNT` 命令作用于多个键时，
返回所有给定 HyperLogLog 的并集的近似基数，
这个近似基数是通过将所有给定 HyperLogLog 合并至一个临时 HyperLogLog 来计算得出的。

..
    The HyperLogLog data structure can be used in order to count unique elements in a set 
    using just a small constant amount of memory, 
    specifically 12k bytes for every HyperLogLog 
    (plus a few bytes for the key itself).

通过 HyperLogLog 数据结构，
用户可以使用少量固定大小的内存，
来储存集合中的唯一元素
（每个 HyperLogLog 只需使用 12k 字节内存，以及几个字节的内存来储存键本身）。

..
    The returned cardinality of the observed set is not exact, 
    but approximated with a standard error of 0.81%.

命令返回的可见集合（observed set）基数并不是精确值，
而是一个带有 0.81% 标准错误（standard error）的近似值。

..
    For example 
    in order to take the count of all the unique search queries performed in a day, 
    a program needs to call PFADD every time a query is processed. 
    The estimated number of unique queries can be retrieved with PFCOUNT at any time.

举个例子，
为了记录一天会执行多少次各不相同的搜索查询，
一个程序可以在每次执行搜索查询时调用一次 :ref:`PFADD` ，
并通过调用 :ref:`PFCOUNT` 命令来获取这个记录的近似结果。

..
    Note: as a side effect of calling this function, 
          it is possible that the HyperLogLog is modified, 
          since the last 8 bytes encode the latest computed cardinality for caching purposes. 
          So PFCOUNT is technically a write command.


**可用版本：**
    >= 2.8.9

**时间复杂度：**
    当命令作用于单个 HyperLogLog 时，
    复杂度为 O(1) ，
    并且具有非常低的平均常数时间。
    当命令作用于 N 个 HyperLogLog 时，
    复杂度为 O(N) ，
    常数时间也比处理单个 HyperLogLog 时要大得多。

..
    O(1) with every small average constant times when called with a single key. 
    O(N) with N being the number of keys, 
    and much bigger constant times, when called with multiple keys.

**返回值：**
    整数回复：
    给定 HyperLogLog 包含的唯一元素的近似数量。
    
..  通过 :ref:`PFADD` 命令记录的唯一元素的近似数量。

..
    Integer reply, specifically:
    The approximated number of unique elements observed via PFADD.

::


    redis> PFADD  databases  "Redis"  "MongoDB"  "MySQL"
    (integer) 1

    redis> PFCOUNT  databases
    (integer) 3

    redis> PFADD  databases  "Redis"    # Redis 已经存在，不必对估计数量进行更新
    (integer) 0

    redis> PFCOUNT  databases    # 元素估计数量没有变化
    (integer) 3

    redis> PFADD  databases  "PostgreSQL"    # 添加一个不存在的元素
    (integer) 1

    redis> PFCOUNT  databases    # 估计数量增一
    4

..
    Performances
    ---------------

    When PFCOUNT is called with a single key, 
    performances as excellent 
    even if in theory constant times to process a dense HyperLogLog are high. 
    This is possible 
    because the PFCOUNT uses caching in order to remember the cardinality previously computed, 
    that rarely changes because most PFADD operations will not update any register. 
    Hundreds of operations per second are possible.

    When PFCOUNT is called with multiple keys, 
    an on-the-fly merge of the HyperLogLogs is performed, 
    which is slow, 
    moreover the cardinality of the union can't be cached, 
    so when used with multiple keys PFCOUNT may take a time 
    in the order of magnitude of the millisecond, and should be not abused.

    The user should take in mind that 
    single-key and multiple-keys executions of this command 
    are semantically different and have different performances.

..
    HyperLogLog representation
    -------------------------------

    Redis HyperLogLogs are represented using a double representation: 
    the sparse representation suitable for HLLs counting a small number of elements 
    (resulting in a small number of registers set to non-zero value), 
    and a dense representation suitable for higher cardinalities. 
    Redis automatically switches from the sparse to the dense representation when needed.

    The sparse representation uses a run-length encoding optimized to store efficiently a big number of registers set to zero. 
    The dense representation is a Redis string of 12288 bytes in order to store 16384 6-bit counters. 
    The need for the double representation comes from the fact that using 12k 
    (which is the dense representation memory requirement) 
    to encode just a few registers for smaller cardinalities is extremely suboptimal.

    Both representations are prefixed with a 16 bytes header, 
    that includes a magic, 
    an encoding / version fiend, 
    and the cached cardinality estimation computed, 
    stored in little endian format 
    (the most significant bit is 1 
    if the estimation is invalid 
    since the HyperLogLog was updated 
    since the cardinality was computed).

    The HyperLogLog, 
    being a Redis string, 
    can be retrieved with GET and restored with SET. 
    Calling PFADD, PFCOUNT or PFMERGE commands with a corrupted HyperLogLog is never a problem, 
    it may return random values 
    but does not affect the stability of the server. 
    Most of the times when corrupting a sparse representation, 
    the server recognizes the corruption and returns an error.

    The representation is neutral from the point of view of the processor word size and endianess, 
    so the same representation is used by 32 bit and 64 bit processor, 
    big endian or little endian.

    More details about the Redis HyperLogLog implementation can be found in this blog post. 
    The source code of the implementation in the hyperloglog.c file is also easy to read and understand, 
    and includes a full specification for the exact encoding 
    used for the sparse and dense representations.
