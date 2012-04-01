.. _subscribe:

SUBSCRIBE
==========

**SUBSCRIBE channel [channel ...]**

订阅给定的一个或多个频道的信息。

**可用版本：**
    >= 2.0.0

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
