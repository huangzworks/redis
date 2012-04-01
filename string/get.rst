.. _get:

GET
====

**GET key**
    
返回 ``key`` 所关联的字符串值。

如果 ``key`` 不存在那么返回特殊值 ``nil`` 。

假如 ``key`` 储存的值不是字符串类型，返回一个错误，因为 `GET`_ 只能用于处理字符串值。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 当 ``key`` 不存在时，返回 ``nil`` ，否则，返回 ``key`` 的值。
    | 如果 ``key`` 不是字符串类型，那么返回一个错误。

::

    # 对不存在的 key 或字符串类型 key 进行 GET

    redis> GET db
    (nil)
    
    redis> SET db redis
    OK

    redis> GET db
    "redis"


    # 对不是字符串类型的 key 进行 GET

    redis> DEL db
    (integer) 1

    redis> LPUSH db redis mongodb mysql
    (integer) 3

    redis> GET db
    (error) ERR Operation against a key holding the wrong kind of value
