.. _publish:

PUBLISH
=========

**PUBLISH channel message**

将信息 ``message`` 发送到指定的频道 ``channel`` 。

**可用版本：**
    >= 2.0.0

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
