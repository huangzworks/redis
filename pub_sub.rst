.. _pub_sub_struct:

发布/订阅(Pub/Sub)
*********************

.. _publish:

PUBLISH
=========

.. function:: PUBLISH channel message

将信息 ``message`` 发送到指定的频道 ``channel`` 。

**时间复杂度：**
    O(N+M)，其中 ``N`` 是频道 ``channel`` 的订阅者数量，而 ``M`` 则是使用模式订阅(subscribed patterns)的客户端的数量。

**返回值：**
    接收到信息 ``message`` 的订阅者数量。

::

    # 对没有订阅者的频道发送信息

    redis> publish bad_channel "can any body hear me?"
    (integer) 0

    # 向有一个订阅者的频道发送信息

    redis> publish msg "good morning"
    (integer) 1

    # 向有多个订阅者的频道发送信息

    redis> publish chat_room "hello~ everyone"
    (integer) 3


.. _subscribe:

SUBSCRIBE
==========

.. function:: SUBSCRIBE channel [channel ...]

订阅给定频道的信息。

**时间复杂度：**
    O(N)，其中 ``N`` 是订阅的频道的数量。

**返回值：**
    接收到的信息(请参见下面的代码说明)。

::

    # 订阅 msg 和 chat_room 两个频道

    # 1 - 6 行是执行 subscribe 之后的反馈信息
    # 第 7 - 9 行才是接收到的第一条信息
    # 第 10 - 12 行是第二条

    redis> subscribe msg chat_room
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"       # 返回值的类型：显示订阅成功
    2) "msg"             # 订阅的频道名字
    3) (integer) 1       # 目前已订阅的频道数量

    1) "subscribe"
    2) "chat_room"
    3) (integer) 2

    1) "message"         # 返回值的类型：信息
    2) "msg"             # 来源(从那个频道发送过来)
    3) "hello moto"      # 信息内容

    1) "message"
    2) "chat_room"
    3) "testing...haha"


.. _psubscribe:

PSUBSCRIBE
===========

.. function:: PSUBSCRIBE pattern [pattern ...]

订阅符合给定模式的频道。

每个模式以 ``*`` 作为匹配符，比如 ``huangz*`` 匹配所有以 ``huangz`` 开头的频道( ``huangzmsg`` 、 ``huangz-blog`` 、 ``huangz.tweets`` 等等)， ``news.*`` 匹配所有以 ``news.`` 开头的频道( ``news.it`` 、 ``news.global.today`` 等等)，诸如此类。

**时间复杂度：**
    O(N)， ``N`` 是订阅的模式的数量。

**返回值：**
    接收到的信息(请参见下面的代码说明)。

::

    # 订阅 news.* 和 tweet.* 两个模式

    # 第 1 - 6 行是执行 psubscribe 之后的反馈信息
    # 第 7 - 10 才是接收到的第一条信息
    # 第 11 - 14 是第二条
    # 以此类推。。。

    redis> psubscribe news.* tweet.*
    Reading messages... (press Ctrl-C to quit)
    1) "psubscribe"     # 返回值的类型：显示订阅成功
    2) "news.*"         # 订阅的模式
    3) (integer) 1      # 目前已订阅的模式的数量

    1) "psubscribe"
    2) "tweet.*"
    3) (integer) 2

    1) "pmessage"               # 返回值的类型：信息
    2) "news.*"                 # 信息匹配的模式
    3) "news.it"                # 信息本身的目标频道
    4) "Google buy Motorola"    # 信息的内容

    1) "pmessage"
    2) "tweet.*"
    3) "tweet.huangz"
    4) "hello"

    1) "pmessage"
    2) "tweet.*"
    3) "tweet.joe"
    4) "@huangz morning"

    1) "pmessage"
    2) "news.*"
    3) "news.life"
    4) "An apple a day, keep doctors away"


.. _unsubscribe:

UNSUBSCRIBE
=============

.. warning:: 此命令在新版 Redis 中似乎已经被废弃？

.. _punsubscribe:

PUNSUBSCRIBE
===============

.. warning:: 此命令在新版 Redis 中似乎已经被废弃？
