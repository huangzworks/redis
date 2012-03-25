.. _renamenx:

RENAMENX
=========

.. function:: RENAMENX key newkey

当且仅当\ ``newkey``\ 不存在时，将\ ``key``\ 改为\ ``newkey``\ 。

出错的情况和\ `RENAME`_\ 一样(\ ``key``\ 不存在时报错)。

**时间复杂度：**
    O(1)

**返回值：**
    | 修改成功时，返回\ ``1``\ 。
    | 如果\ ``newkey``\ 已经存在，返回\ ``0``\ 。

::

    # 情况1：newkey不存在，成功

    redis> SET player "MPlyaer"
    OK

    redis> EXISTS best_player
    (integer) 0

    redis> RENAMENX player best_player
    (integer) 1


    # 情况2：newkey存在时，失败

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


