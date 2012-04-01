.. _hincrbyfloat:

HINCRBYFLOAT
==============

**HINCRBYFLOAT key field increment**

为哈希表 ``key`` 中的域 ``field`` 加上浮点数增量 ``increment`` 。

如果哈希表中没有域 ``field`` ，那么 `HINCRBYFLOAT`_ 会先将域 ``field`` 的值设为 ``0`` ，然后再执行加法操作。

如果键 ``key`` 不存在，那么 `HINCRBYFLOAT`_ 会先创建一个哈希表，再创建域 ``field`` ，最后再执行加法操作。

当以下任意一个条件发生时，返回一个错误：

- 域 ``field`` 的值不是字符串类型(因为 redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
- 域 ``field`` 当前的值或给定的增量 ``increment`` 不能解释(parse)为双精度浮点数(double precision floating point number)

`HINCRBYFLOAT`_ 命令的详细功能和 :ref:`INCRBYFLOAT` 命令类似，请查看 :ref:`INCRBYFLOAT` 命令获取更多相关信息。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(1)

**返回值：**
    执行加法操作之后 ``field`` 域的值。

::

    # 值和增量都是普通小数

    redis> HSET mykey field 10.50
    (integer) 1
    redis> HINCRBYFLOAT mykey field 0.1
    "10.6"


    # 值和增量都是指数符号

    redis> HSET mykey field 5.0e3
    (integer) 0
    redis> HINCRBYFLOAT mykey field 2.0e2
    "5200"


    # 对不存在的键执行 HINCRBYFLOAT

    redis> EXISTS price
    (integer) 0
    redis> HINCRBYFLOAT price milk 3.5
    "3.5"
    redis> HGETALL price
    1) "milk"
    2) "3.5"


    # 对不存在的域进行 HINCRBYFLOAT

    redis> HGETALL price
    1) "milk"
    2) "3.5"
    redis> HINCRBYFLOAT price coffee 4.5   # 新增 coffee 域
    "4.5"
    redis> HGETALL price
    1) "milk"
    2) "3.5"
    3) "coffee"
    4) "4.5"
