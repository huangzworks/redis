.. _renamenx:

RENAMENX
=========

**RENAMENX key newkey**

当且仅当 ``newkey`` 不存在时，将 ``key`` 改名为 ``newkey`` 。

当 ``key`` 不存在时，返回一个错误。

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
