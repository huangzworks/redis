.. _set:

SET
===

.. function:: SET key value

将字符串值\ ``value``\ 关联到\ ``key``\ 。

如果\ ``key``\ 已经持有其他值，\ `SET`_\ 就覆写旧值，无视类型。

**时间复杂度：**
    O(1)

**返回值：**
    总是返回\ ``OK``\ ，因为\ `SET`_\ 不可能失败。

::

    # 情况1：对字符串类型的 key 进行 SET

    redis> SET apple www.apple.com
    OK

    redis> GET apple
    "www.apple.com"


    # 情况2：对非字符串类型的 key 进行 SET

    redis> LPUSH greet_list "hello"  # 建立一个列表
    (integer) 1

    redis> TYPE greet_list
    list

    redis> SET greet_list "yooooooooooooooooo"   # 覆盖列表类型
    OK

    redis> TYPE greet_list
    string


