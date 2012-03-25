.. _randomkey:

RANDOMKEY
==========

**RANDOMKEY**

从当前数据库中随机返回(不删除)一个\ ``key``\ 。

**时间复杂度：**
    O(1)

**返回值：**
    | 当数据库不为空时，返回一个\ ``key``\ 。
    | 当数据库为空时，返回\ ``nil``\ 。

:: 

    # 情况1：数据库不为空

    redis> mset fruit "apple" drink "beer" food "cookies"   # 设置多个key
    OK

    redis> randomkey
    "fruit"

    redis> randomkey
    "food"

    redis> keys *    # 查看数据库内所有key，证明RANDOMKEY并不删除key
    1) "food"
    2) "drink"
    3) "fruit"


    # 情况2：数据库为空

    redis> flushdb  # 删除当前数据库所有key
    OK

    redis> randomkey
    (nil)
