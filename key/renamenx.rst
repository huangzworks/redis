.. _renamenx:

RENAMENX
=========

**RENAMENX key newkey**

当且仅当 ``newkey`` 不存在时，将 ``key`` 改为 ``newkey`` 。

出错的情况和 `RENAME`_ 一样( ``key`` 不存在时报错)。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 修改成功时，返回 ``1`` 。
    | 如果 ``newkey`` 已经存在，返回 ``0`` 。

::

    # newkey 不存在，改名成功

    redis> SET player "MPlyaer"
    OK

    redis> EXISTS best_player
    (integer) 0

    redis> RENAMENX player best_player
    (integer) 1


    # newkey存在时，失败

    redis> SET animal "bear"
    OK

    redis> SET favorite_animal "butterfly"
    OK

    redis> RENAMENX animal favorite_animal
    (integer) 0

    redis> get animal
    "bear"

    redis> get favorite_animal
    "butterfly"
