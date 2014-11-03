.. _config_set:

CONFIG SET
============

**CONFIG SET parameter value**

`CONFIG SET`_ 命令可以动态地调整 Redis 服务器的配置(configuration)而无须重启。

你可以使用它修改配置参数，或者改变 Redis 的持久化(Persistence)方式。

`CONFIG SET`_ 可以修改的配置参数可以使用命令 ``CONFIG GET *`` 来列出，所有被 `CONFIG SET`_ 修改的配置参数都会立即生效。

关于 `CONFIG SET`_ 命令的更多消息，请参见命令 :ref:`config_get` 的说明。

关于如何使用 `CONFIG SET`_ 命令修改 Redis 持久化方式，请参见 `Redis Persistence <http://redis.io/topics/persistence>`_ 。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    不明确

**返回值：**
    当设置成功时返回 ``OK`` ，否则返回一个错误。

::

    redis> CONFIG GET slowlog-max-len
    1) "slowlog-max-len"
    2) "1024"

    redis> CONFIG SET slowlog-max-len 10086
    OK

    redis> CONFIG GET slowlog-max-len
    1) "slowlog-max-len"
    2) "10086"


