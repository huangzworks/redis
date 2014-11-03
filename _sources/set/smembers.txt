.. _smembers:

SMEMBERS
=========

**SMEMBERS key**

返回集合 ``key`` 中的所有成员。

不存在的 ``key`` 被视为空集合。

**可用版本：**
    >= 1.0.0

**时间复杂度:**
    O(N)， ``N`` 为集合的基数。

**返回值:**
    集合中的所有成员。

::

    # key 不存在或集合为空

    redis> EXISTS not_exists_key
    (integer) 0

    redis> SMEMBERS not_exists_key
    (empty list or set)


    # 非空集合

    redis> SADD language Ruby Python Clojure
    (integer) 3

    redis> SMEMBERS language
    1) "Python"
    2) "Ruby"
    3) "Clojure"
