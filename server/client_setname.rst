.. _client_setname:

CLIENT SETNAME
================

**CLIENT SETNAME connection-name**

为当前连接分配一个名字。

这个名字会显示在 :ref:`client_list` 命令的结果中，
用于识别当前正在与服务器进行连接的客户端。

举个例子，
在使用 Redis 构建队列（queue）时，
可以根据连接负责的任务（role），
为信息生产者（producer）和信息消费者（consumer）分别设置不同的名字。

名字使用 Redis 的字符串类型来保存，
最大可以占用 512 MB 。
另外，
为了避免和 :ref:`client_list` 命令的输出格式发生冲突，
名字里不允许使用空格。

要移除一个连接的名字，
可以将连接的名字设为空字符串 ``""`` 。

使用 :ref:`client_getname` 命令可以取出连接的名字。

新创建的连接默认是没有名字的。

.. tip:: 

    在 Redis 应用程序发生连接泄漏时，为连接设置名字是一种很好的 debug 手段。

**可用版本**
    >= 2.6.9

**时间复杂度**
    O(1)

**返回值**
    设置成功时返回 ``OK`` 。

::

    # 新连接默认没有名字

    redis 127.0.0.1:6379> CLIENT GETNAME
    (nil)

    # 设置名字

    redis 127.0.0.1:6379> CLIENT SETNAME hello-world-connection
    OK

    # 返回名字

    redis 127.0.0.1:6379> CLIENT GETNAME
    "hello-world-connection"

    # 在客户端列表中查看

    redis 127.0.0.1:6379> CLIENT LIST
    addr=127.0.0.1:36851 
    fd=5 
    name=hello-world-connection     # <- 名字
    age=51 
    ...

    # 清除名字

    redis 127.0.0.1:6379> CLIENT SETNAME        # 只用空格是不行的！
    (error) ERR Syntax error, try CLIENT (LIST | KILL ip:port)

    redis 127.0.0.1:6379> CLIENT SETNAME ""     # 必须双引号显示包围
    OK

    redis 127.0.0.1:6379> CLIENT GETNAME        # 清除完毕
    (nil)
