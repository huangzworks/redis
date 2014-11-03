.. _incrbyfloat:

INCRBYFLOAT
===============

**INCRBYFLOAT key increment**

为 ``key`` 中所储存的值加上浮点数增量 ``increment`` 。

如果 ``key`` 不存在，那么 `INCRBYFLOAT`_ 会先将 ``key`` 的值设为 ``0`` ，再执行加法操作。

如果命令执行成功，那么 ``key`` 的值会被更新为（执行加法之后的）新值，并且新值会以字符串的形式返回给调用者。

无论是 ``key`` 的值，还是增量 ``increment`` ，都可以使用像 ``2.0e7`` 、 ``3e5`` 、 ``90e-2`` 那样的指数符号(exponential notation)来表示，但是，\ **执行 INCRBYFLOAT 命令之后的值**\ 总是以同样的形式储存，也即是，它们总是由一个数字，一个（可选的）小数点和一个任意位的小数部分组成（比如 ``3.14`` 、 ``69.768`` ，诸如此类)，小数部分尾随的 ``0`` 会被移除，如果有需要的话，还会将浮点数改为整数（比如 ``3.0`` 会被保存成 ``3`` ）。

除此之外，无论加法计算所得的浮点数的实际精度有多长， `INCRBYFLOAT`_ 的计算结果也最多只能表示小数点的后十七位。

当以下任意一个条件发生时，返回一个错误：

- ``key`` 的值不是字符串类型(因为 Redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
- ``key`` 当前的值或者给定的增量 ``increment`` 不能解释(parse)为双精度浮点数(double precision floating point number）

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(1)

**返回值：**
    执行命令之后 ``key`` 的值。

::

    # 值和增量都不是指数符号

    redis> SET mykey 10.50
    OK

    redis> INCRBYFLOAT mykey 0.1
    "10.6"


    # 值和增量都是指数符号

    redis> SET mykey 314e-2
    OK

    redis> GET mykey                # 用 SET 设置的值可以是指数符号
    "314e-2"

    redis> INCRBYFLOAT mykey 0      # 但执行 INCRBYFLOAT 之后格式会被改成非指数符号
    "3.14"


    # 可以对整数类型执行

    redis> SET mykey 3
    OK

    redis> INCRBYFLOAT mykey 1.1
    "4.1"


    # 后跟的 0 会被移除

    redis> SET mykey 3.0
    OK

    redis> GET mykey                                    # SET 设置的值小数部分可以是 0
    "3.0"

    redis> INCRBYFLOAT mykey 1.000000000000000000000    # 但 INCRBYFLOAT 会将无用的 0 忽略掉，有需要的话，将浮点变为整数
    "4"

    redis> GET mykey     
    "4"
