.. _pfadd:

PFADD
===========

**PFADD key element [element ...]**

..
    Adds all the element arguments to the HyperLogLog data structure 
    stored at the variable name specified as first argument.

将任意数量的元素添加到指定的 HyperLogLog 里面。

..
    As a side effect of this command 
    the HyperLogLog internals may be updated 
    to reflect a different estimation of the number of unique items added so far 
    (the cardinality of the set).

作为这个命令的副作用，
HyperLogLog 内部可能会被更新，
以便反映一个不同的唯一元素估计数量（也即是集合的基数）。

..
    If the approximated cardinality estimated by the HyperLogLog changed after executing the command, 
    PFADD returns 1, 
    otherwise 0 is returned. 
    The command automatically creates an empty HyperLogLog structure 
    (that is, a Redis String of a specified length and with a given encoding)
    if the specified key does not exist.

如果 HyperLogLog 估计的近似基数（approximated cardinality）在命令执行之后出现了变化，
那么命令返回 ``1`` ，
否则返回 ``0`` 。
如果命令执行时给定的键不存在，
那么程序将先创建一个空的 HyperLogLog 结构，
然后再执行命令。

..
    To call the command without elements but just the variable name is valid, 
    this will result into no operation performed if the variable already exists, 
    or just the creation of the data structure if the key does not exist 
    (in the latter case 1 is returned).

调用 :ref:`PFADD` 命令时可以只给定键名而不给定元素：

- 如果给定键已经是一个 HyperLogLog ，
  那么这种调用不会产生任何效果；

- 但如果给定的键不存在，
  那么命令会创建一个空的 HyperLogLog ，
  并向客户端返回 ``1`` 。

..  
    For an introduction to HyperLogLog data structure check the PFCOUNT command page.

要了解更多关于 HyperLogLog 数据结构的介绍知识，
请查阅 :ref:`PFCOUNT` 命令的文档。

**可用版本：**
    >= 2.8.9

**时间复杂度：**
    每添加一个元素的复杂度为 O(1) 。
    
**返回值：**
    整数回复：
    如果 HyperLogLog 的内部储存被修改了，
    那么返回 1 ，
    否则返回 0 。

..
    Integer reply, specifically:
    1 if at least 1 HyperLogLog internal register was altered. 0 otherwise.

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
