.. _sunionstore:

SUNIONSTORE
============

**SUNIONSTORE destination key [key ...]**

此命令等同于\ `SUNION`_\，但它将结果保存到\ ``destination``\ 集合，而不是简单地返回结果集。

如果\ ``destination``\ 已经存在，则将其覆盖。

\ ``destination``\ 可以是\ ``key``\ 本身。

**时间复杂度:**
    O(N)，\ ``N``\ 是所有给定集合的成员数量之和。

**返回值:**
    结果集中的元素数量。

::

    redis> SMEMBERS ms_sites
    1) "microsoft.com"
    2) "skype.com"

    redis> SMEMBERS google_sites
    1) "youtube.com"
    2) "google.com"

    redis> SUNIONSTORE google_and_ms_sites ms_sites google_sites
    (integer) 4

    redis> SMEMBERS google_and_ms_sites
    1) "microsoft.com"
    2) "skype.com"
    3) "google.com"
    4) "youtube.com"



