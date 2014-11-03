.. _append:

APPEND
======

**APPEND key value**

如果 ``key`` 已经存在并且是一个字符串， `APPEND`_ 命令将 ``value`` 追加到 ``key`` 原来的值的末尾。

如果 ``key`` 不存在， `APPEND`_ 就简单地将给定 ``key`` 设为 ``value`` ，就像执行 ``SET key value`` 一样。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    平摊O(1)

**返回值：**
    追加 ``value`` 之后， ``key`` 中字符串的长度。

::

    # 对不存在的 key 执行 APPEND

    redis> EXISTS myphone               # 确保 myphone 不存在
    (integer) 0

    redis> APPEND myphone "nokia"       # 对不存在的 key 进行 APPEND ，等同于 SET myphone "nokia"
    (integer) 5                         # 字符长度


    # 对已存在的字符串进行 APPEND

    redis> APPEND myphone " - 1110"     # 长度从 5 个字符增加到 12 个字符
    (integer) 12  

    redis> GET myphone 
    "nokia - 1110"

模式：时间序列(Time series)
------------------------------

`APPEND`_ 可以为一系列定长(fixed-size)数据(sample)提供一种紧凑的表示方式，通常称之为时间序列。

每当一个新数据到达的时候，执行以下命令：

::

    APPEND timeseries "fixed-size sample"
    
然后可以通过以下的方式访问时间序列的各项属性：

- :ref:`STRLEN` 给出时间序列中数据的数量
- :ref:`GETRANGE` 可以用于随机访问。只要有相关的时间信息的话，我们就可以在 Redis 2.6 中使用 Lua 脚本和 :ref:`GETRANGE` 命令实现二分查找。
- :ref:`SETRANGE` 可以用于覆盖或修改已存在的的时间序列。

这个模式的唯一缺陷是我们只能增长时间序列，而不能对时间序列进行缩短，因为 Redis 目前还没有对字符串进行修剪(tirm)的命令，但是，不管怎么说，这个模式的储存方式还是可以节省下大量的空间。

.. note:: 可以考虑使用 UNIX 时间戳作为时间序列的键名，这样一来，可以避免单个 ``key`` 因为保存过大的时间序列而占用大量内存，另一方面，也可以节省下大量命名空间。

下面是一个时间序列的例子：

::

    redis> APPEND ts "0043"
    (integer) 4

    redis> APPEND ts "0035"
    (integer) 8

    redis> GETRANGE ts 0 3
    "0043"

    redis> GETRANGE ts 4 7
    "0035"
