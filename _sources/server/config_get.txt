.. _config_get:

CONFIG GET
=============

**CONFIG GET parameter**

`CONFIG GET`_ 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters)，在 Redis 2.4 版本中， 有部分参数没有办法用 ``CONFIG GET`` 访问，但是在最新的 Redis 2.6 版本中，所有配置参数都已经可以用 ``CONFIG GET`` 访问了。

`CONFIG GET`_ 接受单个参数 ``parameter`` 作为搜索关键字，查找所有匹配的配置参数，其中参数和值以“键-值对”(key-value pairs)的方式排列。

比如执行 ``CONFIG GET s*`` 命令，服务器就会返回所有以 ``s`` 开头的配置参数及参数的值：

::

    redis> CONFIG GET s*
    1) "save"                       # 参数名：save
    2) "900 1 300 10 60 10000"      # save 参数的值
    3) "slave-serve-stale-data"     # 参数名： slave-serve-stale-data
    4) "yes"                        # slave-serve-stale-data 参数的值
    5) "set-max-intset-entries"     # ...
    6) "512"
    7) "slowlog-log-slower-than"
    8) "1000"
    9) "slowlog-max-len"
    10) "1000"

如果你只是寻找特定的某个参数的话，你当然也可以直接指定参数的名字：

::

    redis> CONFIG GET slowlog-max-len
    1) "slowlog-max-len"
    2) "1000"

使用命令 ``CONFIG GET *`` ，可以列出 ``CONFIG GET`` 命令支持的所有参数：

::

    redis> CONFIG GET *
    1) "dir"
    2) "/var/lib/redis"
    3) "dbfilename"
    4) "dump.rdb"
    5) "requirepass"
    6) (nil)
    7) "masterauth"
    8) (nil)
    9) "maxmemory"
    10) "0"
    11) "maxmemory-policy"
    12) "volatile-lru"
    13) "maxmemory-samples"
    14) "3"
    15) "timeout"
    16) "0"
    17) "appendonly"
    18) "no"
    # ...
    49) "loglevel"
    50) "verbose"


所有被 ``CONFIG SET`` 所支持的配置参数都可以在配置文件 redis.conf 中找到，不过 ``CONFIG GET`` 和 ``CONFIG SET`` 使用的格式和 redis.conf 文件所使用的格式有以下两点不同：

- | ``10kb`` 、 ``2gb`` 这些在配置文件中所使用的储存单位缩写，不可以用在 ``CONFIG`` 命令中， ``CONFIG SET`` 的值只能通过数字值显式地设定。
  | 
  | 像 ``CONFIG SET xxx 1k`` 这样的命令是错误的，正确的格式是 ``CONFIG SET xxx 1000`` 。

- | ``save`` 选项在 redis.conf 中是用多行文字储存的，但在 ``CONFIG GET`` 命令中，它只打印一行文字。
  |
  | 以下是 ``save`` 选项在 redis.conf 文件中的表示：
  |
  | ``save 900 1``
  | ``save 300 10``
  | ``save 60 10000``
  |
  | 但是 ``CONFIG GET`` 命令的输出只有一行：
  |
  | ``redis> CONFIG GET save``
  | ``1) "save"``
  | ``2) "900 1 300 10 60 10000"``
  | 
  | 上面 ``save`` 参数的三个值表示：在 900 秒内最少有 1 个 key 被改动，或者 300 秒内最少有 10 个 key 被改动，又或者 60 秒内最少有 1000 个 key 被改动，以上三个条件随便满足一个，就触发一次保存操作。

**可用版本：**
    >= 2.0.0

**时间复杂度：**
    不明确

**返回值：**
    给定配置参数的值。
