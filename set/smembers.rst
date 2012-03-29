.. _smembers:

SMEMBERS
=========

**SMEMBERS key**

返回集合\ ``key``\ 中的所有成员。

**时间复杂度:**
    O(N)，\ ``N``\ 为集合的基数。

**返回值:**
    集合中的所有成员。

::

    # 情况1：空集合

    redis> EXISTS not_exists_key    # 不存在的 key 视为空集合
    (integer) 0

    redis> SMEMBERS not_exists_key
    (empty list or set)

    
    # 情况2：非空集合

    redis> SADD programming_language python
    (integer) 1

    redis> SADD programming_language ruby
    (integer) 1

    redis> SADD programming_language c
    (integer) 1

    redis> SMEMBERS programming_language
    1) "c"
    2) "ruby"
    3) "python"



