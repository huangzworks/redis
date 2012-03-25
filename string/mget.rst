.. _mget:

MGET
=====
.. function:: MGET key [key ...] 

返回所有(一个或多个)给定\ ``key``\ 的值。

如果某个指定\ ``key``\ 不存在，那么返回特殊值\ ``nil``\ 。因此，该命令永不失败。

**时间复杂度:**
    O(1)
                                        
**返回值：**
    一个包含所有给定\ ``key``\ 的值的列表。

::

    redis> MSET name huangz twitter twitter.com/huangz1990  #用MSET一次储存多个值
    OK

    redis> MGET name twitter
    1) "huangz"
    2) "twitter.com/huangz1990"

    redis> EXISTS fake_key
    (integer) 0

    redis> MGET name fake_key  # 当MGET中有不存在key的情况
    1) "huangz"
    2) (nil)


