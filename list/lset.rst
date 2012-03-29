.. _lset:

LSET
=======

**LSET key index value**

将列表\ ``key``\ 下标为\ ``index``\ 的元素的值甚至为\ ``value``\ 。

更多信息请参考\ `LINDEX`_\ 操作。 

当\ ``index``\ 参数超出范围，或对一个空列表(\ ``key``\ 不存在)进行\ `LSET`_\ 时，返回一个错误。

**时间复杂度：**
    | 对头元素或尾元素进行\ `LSET`_\ 操作，复杂度为 O(1)。
    | 其他情况下，为 O(N)，\ ``N``\ 为列表的长度。

**返回值：**
    操作成功返回\ ``ok``\ ，否则返回错误信息。

::

    # 情况1：对空列表(key 不存在)进行 LSET

    redis> EXISTS list
    (integer) 0

    redis> LSET list 0 item
    (error) ERR no such key


    # 情况2：对非空列表进行 LSET

    redis> LPUSH job "cook food"
    (integer) 1

    redis> LRANGE job 0 0
    1) "cook food"

    redis> LSET job 0 "play game"
    OK

    redis> LRANGE job  0 0
    1) "play game"


    # 情况3：index 超出范围

    redis> LLEN list # 列表长度为 1
    (integer) 1

    redis> LSET list 3 'out of range'
    (error) ERR index out of range


