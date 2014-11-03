.. _select:

SELECT
========

**SELECT index**

切换到指定的数据库，数据库索引号 ``index`` 用数字值指定，以 ``0`` 作为起始索引值。

默认使用 ``0`` 号数据库。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    ``OK``

::

    redis> SET db_number 0         # 默认使用 0 号数据库
    OK

    redis> SELECT 1                # 使用 1 号数据库
    OK

    redis[1]> GET db_number        # 已经切换到 1 号数据库，注意 Redis 现在的命令提示符多了个 [1]
    (nil)

    redis[1]> SET db_number 1
    OK

    redis[1]> GET db_number
    "1"

    redis[1]> SELECT 3             # 再切换到 3 号数据库
    OK

    redis[3]>                      # 提示符从 [1] 改变成了 [3]
