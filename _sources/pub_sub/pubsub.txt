.. _pubsub:

PUBSUB
=========

**PUBSUB <subcommand> [argument [argument ...]]**

:ref:`PUBSUB` 是一个查看订阅与发布系统状态的内省命令，
它由数个不同格式的子命令组成，
以下将分别对这些子命令进行介绍。

**可用版本：** >= 2.8.0


PUBSUB CHANNELS [pattern]
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

列出当前的活跃频道。

活跃频道指的是那些至少有一个订阅者的频道，
订阅模式的客户端不计算在内。

``pattern`` 参数是可选的：

- 如果不给出 ``pattern`` 参数，那么列出订阅与发布系统中的所有活跃频道。

- 如果给出 ``pattern`` 参数，那么只列出和给定模式 ``pattern`` 相匹配的那些活跃频道。

**复杂度：** O(N) ， ``N`` 为活跃频道的数量（对于长度较短的频道和模式来说，将进行模式匹配的复杂度视为常数）。

**返回值：** 一个由活跃频道组成的列表。

::

    # client-1 订阅 news.it 和 news.sport 两个频道

    client-1> SUBSCRIBE news.it news.sport 
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "news.it"
    3) (integer) 1
    1) "subscribe"
    2) "news.sport"
    3) (integer) 2

    # client-2 订阅 news.it 和 news.internet 两个频道

    client-2> SUBSCRIBE news.it news.internet
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "news.it"
    3) (integer) 1
    1) "subscribe"
    2) "news.internet"
    3) (integer) 2

    # 首先， client-3 打印所有活跃频道
    # 注意，即使一个频道有多个订阅者，它也只输出一次，比如 news.it

    client-3> PUBSUB CHANNELS
    1) "news.sport"
    2) "news.internet"
    3) "news.it"

    # 接下来， client-3 打印那些与模式 news.i* 相匹配的活跃频道
    # 因为 news.sport 不匹配 news.i* ，所以它没有被打印

    redis> PUBSUB CHANNELS news.i*
    1) "news.internet"
    2) "news.it"


PUBSUB NUMSUB [channel-1 ... channel-N]
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

返回给定频道的订阅者数量，
订阅模式的客户端不计算在内。

**复杂度：** O(N) ， ``N`` 为给定频道的数量。

**返回值：**
一个多条批量回复（Multi-bulk reply），回复中包含给定的频道，以及频道的订阅者数量。
格式为：频道 ``channel-1`` ， ``channel-1`` 的订阅者数量，频道 ``channel-2`` ， ``channel-2`` 的订阅者数量，诸如此类。
回复中频道的排列顺序和执行命令时给定频道的排列顺序一致。
不给定任何频道而直接调用这个命令也是可以的，
在这种情况下，
命令只返回一个空列表。

::

    # client-1 订阅 news.it 和 news.sport 两个频道

    client-1> SUBSCRIBE news.it news.sport 
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "news.it"
    3) (integer) 1
    1) "subscribe"
    2) "news.sport"
    3) (integer) 2

    # client-2 订阅 news.it 和 news.internet 两个频道

    client-2> SUBSCRIBE news.it news.internet
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "news.it"
    3) (integer) 1
    1) "subscribe"
    2) "news.internet"
    3) (integer) 2

    # client-3 打印各个频道的订阅者数量

    client-3> PUBSUB NUMSUB news.it news.internet news.sport news.music
    1) "news.it"    # 频道
    2) "2"          # 订阅该频道的客户端数量
    3) "news.internet"
    4) "1"
    5) "news.sport"
    6) "1"
    7) "news.music" # 没有任何订阅者
    8) "0"
    

PUBSUB NUMPAT
^^^^^^^^^^^^^^^^^

返回订阅模式的数量。

注意，
这个命令返回的不是订阅模式的客户端的数量，
而是客户端订阅的所有模式的数量总和。

**复杂度：** O(1) 。

**返回值：** 一个整数回复（Integer reply）。

::

    # client-1 订阅 news.* 和 discount.* 两个模式

    client-1> PSUBSCRIBE news.* discount.*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "news.*"
    3) (integer) 1
    1) "psubscribe"
    2) "discount.*"
    3) (integer) 2

    # client-2 订阅 tweet.* 一个模式

    client-2> PSUBSCRIBE tweet.*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "tweet.*"
    3) (integer) 1

    # client-3 返回当前订阅模式的数量为 3

    client-3> PUBSUB NUMPAT
    (integer) 3

    # 注意，当有多个客户端订阅相同的模式时，相同的订阅也被计算在 PUBSUB NUMPAT 之内
    # 比如说，再新建一个客户端 client-4 ，让它也订阅 news.* 频道

    client-4> PSUBSCRIBE news.*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"
    2) "news.*"
    3) (integer) 1

    # 这时再计算被订阅模式的数量，就会得到数量为 4

    client-3> PUBSUB NUMPAT
    (integer) 4
