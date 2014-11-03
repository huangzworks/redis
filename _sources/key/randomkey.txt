.. _randomkey:

RANDOMKEY
==========

**RANDOMKEY**

从当前数据库中随机返回(不删除)一个 ``key`` 。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 当数据库不为空时，返回一个 ``key`` 。
    | 当数据库为空时，返回 ``nil`` 。

:: 

    # 数据库不为空

    redis> MSET fruit "apple" drink "beer" food "cookies"   # 设置多个 key
    OK

    redis> RANDOMKEY
    "fruit"

    redis> RANDOMKEY
    "food"

    redis> KEYS *    # 查看数据库内所有key，证明 RANDOMKEY 并不删除 key
    1) "food"
    2) "drink"
    3) "fruit"


    # 数据库为空

    redis> FLUSHDB  # 删除当前数据库所有 key
    OK

    redis> RANDOMKEY
    (nil)
