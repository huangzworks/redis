.. _psubscribe:

PSUBSCRIBE
===========

**PSUBSCRIBE pattern [pattern ...]**

订阅一个或多个符合给定模式的频道。

每个模式以 ``*`` 作为匹配符，比如 ``it*`` 匹配所有以 ``it`` 开头的频道( ``it.news`` 、 ``it.blog`` 、 ``it.tweets`` 等等)， ``news.*`` 匹配所有以 ``news.`` 开头的频道( ``news.it`` 、 ``news.global.today`` 等等)，诸如此类。

**可用版本：**
    >= 2.0.0

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
    1) "psubscribe"                  # 返回值的类型：显示订阅成功
    2) "news.*"                      # 订阅的模式
    3) (integer) 1                   # 目前已订阅的模式的数量

    1) "psubscribe"
    2) "tweet.*"
    3) (integer) 2

    1) "pmessage"                    # 返回值的类型：信息
    2) "news.*"                      # 信息匹配的模式
    3) "news.it"                     # 信息本身的目标频道
    4) "Google buy Motorola"         # 信息的内容

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
