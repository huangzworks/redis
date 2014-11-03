.. _incr:

INCR
=====

**INCR key**

将 ``key`` 中储存的数字值增一。

如果 ``key`` 不存在，那么 ``key`` 的值会先被初始化为 ``0`` ，然后再执行 `INCR`_ 操作。 

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

.. note:: 
    这是一个针对字符串的操作，因为 Redis 没有专用的整数类型，所以 key 内储存的字符串被解释为十进制 64 位有符号整数来执行 INCR 操作。 

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    执行 `INCR`_ 命令之后 ``key`` 的值。

::
    
    redis> SET page_view 20
    OK

    redis> INCR page_view
    (integer) 21

    redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
    "21"

模式：计数器
---------------

计数器是 Redis 的原子性自增操作可实现的最直观的模式了，它的想法相当简单：每当某个操作发生时，向 Redis 发送一个 `INCR`_ 命令。

比如在一个 web 应用程序中，如果想知道用户在一年中每天的点击量，那么只要将用户 ID 以及相关的日期信息作为键，并在每次用户点击页面时，执行一次自增操作即可。

比如用户名是 ``peter`` ，点击时间是 2012 年 3 月 22 日，那么执行命令 ``INCR peter::2012.3.22`` 。

可以用以下几种方式扩展这个简单的模式：

- 可以通过组合使用 `INCR`_ 和 :ref:`expire` ，来达到只在规定的生存时间内进行计数(counting)的目的。
- 客户端可以通过使用 :ref:`GETSET` 命令原子性地获取计数器的当前值并将计数器清零，更多信息请参考 :ref:`GETSET` 命令。
- 使用其他自增/自减操作，比如 :ref:`DECR` 和 :ref:`INCRBY` ，用户可以通过执行不同的操作增加或减少计数器的值，比如在游戏中的记分器就可能用到这些命令。

模式：限速器
-------------

限速器是特殊化的计算器，它用于限制一个操作可以被执行的速率(rate)。

限速器的典型用法是限制公开 API 的请求次数，以下是一个限速器实现示例，它将 API 的最大请求数限制在每个 IP 地址每秒钟十个之内：

::

    FUNCTION LIMIT_API_CALL(ip)
    ts = CURRENT_UNIX_TIME()
    keyname = ip+":"+ts
    current = GET(keyname)

    IF current != NULL AND current > 10 THEN
        ERROR "too many requests per second"
    END

    IF current == NULL THEN
        MULTI
            INCR(keyname, 1)
            EXPIRE(keyname, 1)
        EXEC
    ELSE
        INCR(keyname, 1)
    END

    PERFORM_API_CALL()

这个实现每秒钟为每个 IP 地址使用一个不同的计数器，并用 :ref:`EXPIRE` 命令设置生存时间(这样 Redis 就会负责自动删除过期的计数器)。

注意，我们使用事务打包执行 :ref:`INCR` 命令和 :ref:`EXPIRE` 命令，避免引入竞争条件，保证每次调用 API 时都可以正确地对计数器进行自增操作并设置生存时间。

以下是另一个限速器实现：

::

    FUNCTION LIMIT_API_CALL(ip):
    current = GET(ip)
    IF current != NULL AND current > 10 THEN
        ERROR "too many requests per second"
    ELSE
        value = INCR(ip)
        IF value == 1 THEN
            EXPIRE(ip,1)
        END
        PERFORM_API_CALL()
    END

这个限速器只使用单个计数器，它的生存时间为一秒钟，如果在一秒钟内，这个计数器的值大于 ``10`` 的话，那么访问就会被禁止。

这个新的限速器在思路方面是没有问题的，但它在实现方面不够严谨，如果我们仔细观察一下的话，就会发现在 :ref:`INCR` 和 :ref:`EXPIRE` 之间存在着一个竞争条件，假如客户端在执行 :ref:`INCR` 之后，因为某些原因(比如客户端失败)而忘记设置 :ref:`EXPIRE` 的话，那么这个计数器就会一直存在下去，造成每个用户只能访问 ``10`` 次，噢，这简直是个灾难！

要消灭这个实现中的竞争条件，我们可以将它转化为一个 Lua 脚本，并放到 Redis 中运行(这个方法仅限于 Redis 2.6 及以上的版本)：

::
    
    local current
    current = redis.call("incr",KEYS[1])
    if tonumber(current) == 1 then
        redis.call("expire",KEYS[1],1)
    end

通过将计数器作为脚本放到 Redis 上运行，我们保证了 :ref:`INCR` 和 :ref:`EXPIRE` 两个操作的原子性，现在这个脚本实现不会引入竞争条件，它可以运作的很好。

关于在 Redis 中运行 Lua 脚本的更多信息，请参考 :ref:`EVAL` 命令。

还有另一种消灭竞争条件的方法，就是使用 Redis 的列表结构来代替 :ref:`INCR` 命令，这个方法无须脚本支持，因此它在 Redis 2.6 以下的版本也可以运行得很好：

::

    FUNCTION LIMIT_API_CALL(ip)
    current = LLEN(ip)
    IF current > 10 THEN
        ERROR "too many requests per second"
    ELSE
        IF EXISTS(ip) == FALSE
            MULTI
                RPUSH(ip,ip)
                EXPIRE(ip,1)
            EXEC
        ELSE
            RPUSHX(ip,ip)
        END
        PERFORM_API_CALL()
    END

新的限速器使用了列表结构作为容器， :ref:`LLEN` 用于对访问次数进行检查，一个事务包裹着 :ref:`RPUSH` 和 :ref:`EXPIRE` 两个命令，用于在第一次执行计数时创建列表，并正确设置地设置过期时间，最后， :ref:`RPUSHX` 在后续的计数操作中进行增加操作。
