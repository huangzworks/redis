.. _linsert:

LINSERT
=========

**LINSERT key BEFORE|AFTER pivot value**

将值 ``value`` 插入到列表 ``key`` 当中，位于值 ``pivot`` 之前或之后。

当 ``pivot`` 不存在于列表 ``key`` 时，不执行任何操作。

当 ``key`` 不存在时， ``key`` 被视为空列表，不执行任何操作。

如果 ``key`` 不是列表类型，返回一个错误。 

**可用版本：**
    >= 2.2.0

**时间复杂度:**
    O(N)， ``N`` 为寻找 ``pivot`` 过程中经过的元素数量。

**返回值:**
    | 如果命令执行成功，返回插入操作完成之后，列表的长度。
    | 如果没有找到 ``pivot`` ，返回 ``-1`` 。
    | 如果 ``key`` 不存在或为空列表，返回 ``0`` 。

::

    redis> RPUSH mylist "Hello"
    (integer) 1

    redis> RPUSH mylist "World"
    (integer) 2

    redis> LINSERT mylist BEFORE "World" "There"
    (integer) 3

    redis> LRANGE mylist 0 -1
    1) "Hello"
    2) "There"
    3) "World"


    # 对一个非空列表插入，查找一个不存在的 pivot

    redis> LINSERT mylist BEFORE "go" "let's"    
    (integer) -1                                    # 失败


    # 对一个空列表执行 LINSERT 命令

    redis> EXISTS fake_list  
    (integer) 0

    redis> LINSERT fake_list BEFORE "nono" "gogogog"
    (integer) 0                                      # 失败
