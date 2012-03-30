.. _lrem:

LREM
=======

**LREM key count value**

根据参数 ``count`` 的值，移除列表中与参数 ``value`` 相等的元素。
        
``count`` 的值可以是以下几种：

- ``count > 0`` : 从表头开始向表尾搜索，移除与 ``value`` 相等的元素，数量为 ``count`` 。
- ``count < 0`` : 从表尾开始向表头搜索，移除与 ``value`` 相等的元素，数量为 ``count`` 的绝对值。
- ``count = 0`` : 移除表中所有与 ``value`` 相等的值。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(N)， ``N`` 为列表的长度。

**返回值：**
    | 被移除元素的数量。
    | 因为不存在的 ``key`` 被视作空表(empty list)，所以当 ``key`` 不存在时， `LREM`_ 命令总是返回 ``0`` 。

:: 

    # 先创建一个表，内容排列是
    # morning hello morning helllo morning

    redis> LPUSH greet "morning"
    (integer) 1
    redis> LPUSH greet "hello"
    (integer) 2
    redis> LPUSH greet "morning"
    (integer) 3
    redis> LPUSH greet "hello"
    (integer) 4
    redis> LPUSH greet "morning"
    (integer) 5

    redis> LRANGE greet 0 4         # 查看所有元素
    1) "morning"
    2) "hello"
    3) "morning"
    4) "hello"
    5) "morning"

    redis> LREM greet 2 morning     # 移除从表头到表尾，最先发现的两个 morning
    (integer) 2                     # 两个元素被移除

    redis> LLEN greet               # 还剩 3 个元素
    (integer) 3

    redis> LRANGE greet 0 2
    1) "hello"
    2) "hello"
    3) "morning"

    redis> LREM greet -1 morning    # 移除从表尾到表头，第一个 morning
    (integer) 1

    redis> LLEN greet               # 剩下两个元素
    (integer) 2

    redis> LRANGE greet 0 1
    1) "hello"
    2) "hello"

    redis> LREM greet 0 hello      # 移除表中所有 hello
    (integer) 2                    # 两个 hello 被移除

    redis> LLEN greet
    (integer) 0
