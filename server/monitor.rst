.. _monitor:

MONITOR
========

**MONITOR**

实时打印出 Redis 服务器接收到的命令，调试用。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    不明确

**返回值：**
    总是返回 ``OK`` 。

::

    redis> MONITOR
    OK
    1324109476.800290 "MONITOR" # 第一个值是 UNIX 时间戳，之后是执行的命令和参数
    1324109479.632445 "PING"
    1324109486.408230 "SET" "greeting" "hello moto"
    1324109490.800364 "KEYS" "*"
    1324109509.023495 "lrange" "my_book_list" "0" "-1"


