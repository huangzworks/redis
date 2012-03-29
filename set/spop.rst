.. _spop:

SPOP
=====

**SPOP key**

移除并返回集合中的一个随机元素。

**时间复杂度:**
    O(1)

**返回值:**
    | 被移除的随机元素。
    | 当\ ``key``\ 不存在或\ ``key``\ 是空集时，返回\ ``nil``\ 。

::

    redis> SMEMBERS my_sites
    1) "huangz.iteye.com"
    2) "sideeffect.me"
    3) "douban.com/people/i_m_huangz"

    redis> SPOP my_sites
    "huangz.iteye.com"  

    redis> SMEMBERS my_sites
    1) "sideeffect.me"
    2) "douban.com/people/i_m_huang"

.. seealso:: 如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用\ `SRANDMEMBER`_\ 命令。


