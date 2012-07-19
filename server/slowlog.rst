.. _slowlog:

SLOWLOG
==========

**SLOWLOG subcommand [argument]**

**什么是 SLOWLOG**

Slow log 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

**设置 SLOWLOG**

Slow log 的行为由两个配置参数(configuration parameter)指定，可以通过改写 redis.conf 文件或者用 ``CONFIG GET`` 和 ``CONFIG SET`` 命令对它们动态地进行修改。

第一个选项是 ``slowlog-log-slower-than`` ，它决定要对执行时间大于多少微秒(microsecond，1秒 = 1,000,000 微秒)的查询进行记录。

比如执行以下命令将让 slow log 记录所有查询时间大于等于 100 微秒的查询：

``CONFIG SET slowlog-log-slower-than 100``

而以下命令记录所有查询时间大于 1000 微秒的查询：

``CONFIG SET slowlog-log-slower-than 1000``

另一个选项是 ``slowlog-max-len`` ，它决定 slow log *最多*\ 能保存多少条日志， slow log 本身是一个 FIFO 队列，当队列大小超过 ``slowlog-max-len`` 时，最旧的一条日志将被删除，而最新的一条日志加入到 slow log ，以此类推。

以下命令让 slow log 最多保存 1000 条日志：

``CONFIG SET slowlog-max-len 1000``

使用 ``CONFIG GET`` 命令可以查询两个选项的当前值：

::

    redis> CONFIG GET slowlog-log-slower-than
    1) "slowlog-log-slower-than"
    2) "1000"

    redis> CONFIG GET slowlog-max-len
    1) "slowlog-max-len"
    2) "1000"

**查看 slow log**

要查看 slow log ，可以使用 ``SLOWLOG GET`` 或者 ``SLOWLOG GET number`` 命令，前者打印所有 slow log ，最大长度取决于 ``slowlog-max-len`` 选项的值，而 ``SLOWLOG GET number`` 则只打印指定数量的日志。

最新的日志会最先被打印：

::

    # 为测试需要，将 slowlog-log-slower-than 设成了 10 微秒

    redis> SLOWLOG GET
    1) 1) (integer) 12                      # 唯一性(unique)的日志标识符
       2) (integer) 1324097834              # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
       3) (integer) 16                      # 查询执行时间，以微秒为单位
       4) 1) "CONFIG"                       # 执行的命令，以数组的形式排列
          2) "GET"                          # 这里完整的命令是 CONFIG GET slowlog-log-slower-than 
          3) "slowlog-log-slower-than"

    2) 1) (integer) 11
       2) (integer) 1324097825
       3) (integer) 42
       4) 1) "CONFIG"
          2) "GET"
          3) "*"

    3) 1) (integer) 10
       2) (integer) 1324097820
       3) (integer) 11
       4) 1) "CONFIG"
          2) "GET"
          3) "slowlog-log-slower-than"

    # ...

日志的唯一 id 只有在 Redis 服务器重启的时候才会重置，这样可以避免对日志的重复处理(比如你可能会想在每次发现新的慢查询时发邮件通知你)。

**查看当前日志的数量**

使用命令 ``SLOWLOG LEN`` 可以查看当前日志的数量。

请注意这个值和 ``slower-max-len`` 的区别，它们一个是当前日志的数量，一个是允许记录的最大日志的数量。

::

    redis> SLOWLOG LEN
    (integer) 14

**清空日志**

使用命令 ``SLOWLOG RESET`` 可以清空 slow log 。

::

    redis> SLOWLOG LEN
    (integer) 14

    redis> SLOWLOG RESET
    OK

    redis> SLOWLOG LEN
    (integer) 0

**可用版本：**
    >= 2.2.12

**时间复杂度：**
    O(1)

**返回值：**
    取决于不同命令，返回不同的值。
