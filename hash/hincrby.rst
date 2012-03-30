.. _hincrby:

HINCRBY
========

**HINCRBY key field increment**

为哈希表 ``key`` 中的域 ``field`` 的值加上增量 ``increment`` 。

增量也可以为负数，相当于对给定域进行减法操作。

如果 ``key`` 不存在，一个新的哈希表被创建并执行 `HINCRBY`_ 命令。

如果域 ``field`` 不存在，那么在执行命令前，域的值被初始化为 ``0`` 。

对一个储存字符串值的域 ``field`` 执行 `HINCRBY`_ 命令将造成一个错误。

本操作的值被限制在 64 位(bit)有符号数字表示之内。
    
**可用版本：**
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    执行 `HINCRBY`_ 命令之后，哈希表 ``key`` 中域 ``field`` 的值。

::

    # increment 为正数

    redis> HEXISTS counter page_view    # 对空域进行设置
    (integer) 0

    redis> HINCRBY counter page_view 200
    (integer) 200

    redis> HGET counter page_view
    "200"


    # increment 为负数

    redis> HGET counter page_view
    "200"

    redis> HINCRBY counter page_view -50
    (integer) 150

    redis> HGET counter page_view
    "150"


    # 尝试对字符串值的域执行HINCRBY命令
    
    redis> HSET myhash string hello,world       # 设定一个字符串值
    (integer) 1

    redis> HGET myhash string
    "hello,world"

    redis> HINCRBY myhash string 1              # 命令执行失败，错误。
    (error) ERR hash value is not an integer

    redis> HGET myhash string                   # 原值不变
    "hello,world"
