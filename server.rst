.. _server_struct:

服务器(Server)
****************

.. _bgrewriteaof:

BGREWRITEAOF
=============

**BGREWRITEAOF**

异步(Asynchronously)重写 AOF 文件以反应当前数据库的状态。

即使 `BGREWRITEAOF`_ 命令执行失败，旧 AOF 文件中的数据也不会因此丢失或改变。

**时间复杂度：**
    O(N)， ``N`` 为要追加到 AOF 文件中的数据数量。

**返回值：**
    反馈信息。

::
    
    redis> BGREWRITEAOF
    Background append only file rewriting started


.. _bgsave:

BGSAVE
=======

在后台异步保存当前数据库的数据到磁盘。

`BGSAVE`_ 命令执行之后立即返回 ``OK`` ，然后 Redis fork出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

客户端可以通过 `LASTSAVE`_ 命令查看相关信息，判断 `BGSAVE`_ 命令是否执行成功。

**时间复杂度：**
    O(N)， ``N`` 为要保存到数据库中的 key 的数量。

**返回值：**
    反馈信息。

::

    redis> BGSAVE
    Background saving started


.. _save:

SAVE
=====

**SAVE**

同步保存当前数据库的数据到磁盘。

**时间复杂度：**
    O(N)， ``N`` 为要保存到数据库中的 key 的数量。

**返回值：**
    总是返回 ``OK`` 。

::

    redis> SAVE
    OK


.. _lastsave:

LASTSAVE
=========

**LASTSAVE**

返回最近一次 Redis 成功执行保存操作的时间点( `SAVE`_ 、 `BGSAVE`_ 等)，以 UNIX 时间戳格式表示。

**时间复杂度：**
    O(1)

**返回值：**
    一个 UNIX 时间戳。

::

    redis> LASTSAVE
    (integer) 1324043588


.. _dbsize:

DBSIZE
=======

**DBSIZE**

返回当前数据库的 key 的数量。

**时间复杂度：**
    O(1)

**返回值：**
    当前数据库的 key 的数量。

::

    redis> DBSIZE
    (integer) 5

    redis> SET new_key "hello_moto"  # 增加一个 key 试试
    OK

    redis> DBSIZE
    (integer) 6


.. _slaveof:

SLAVEOF
========

.. function:: SLAVEOF host port

`SLAVEOF`_ 命令用于在 Redis 运行时动态地修改复制(replication)功能的行为。

通过执行 ``SLAVEOF host port`` 命令，可以将当前服务器转变为指定服务器的从属服务器(slave server)。

如果当前服务器已经是某个主服务器(master server)的从属服务器，那么执行 ``SLAVEOF host port`` 将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。

另外，对一个从属服务器执行命令 ``SLAVEOF NO ONE`` 将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集\ *不会*\ 被丢弃。

利用“ ``SLAVEOF NO ONE`` 不会丢弃同步所得数据集”这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。

**时间复杂度：**
    | ``SLAVEOF host port`` ，O(N)， ``N`` 为要同步的数据数量。
    | ``SLAVEOF NO ONE`` ， O(1) 。

**返回值：**
    总是返回 ``OK`` 。

::

    redis> SLAVEOF 127.0.0.1 6379
    OK

    redis> SLAVEOF NO ONE
    OK


.. _flushall:

FLUSHALL
=========

**FLUSHALL**

清空整个 Redis 服务器的数据(删除\ *所有数据库的所有 key*\ )。

此命令从不失败。

**时间复杂度：**
    尚未明确

**返回值：**
    总是返回 ``OK`` 。

::

    redis> DBSIZE            # 0 号数据库的 key 数量
    (integer) 9

    redis> SELECT 1          # 切换到 1 号数据库
    OK

    redis[1]> DBSIZE         # 1 号数据库的 key 数量
    (integer) 6

    redis[1]> flushall       # 清空所有数据库的所有 key 
    OK

    redis[1]> DBSIZE         # 不但 1 号数据库被清空了
    (integer) 0

    redis[1]> SELECT 0       # 0 号数据库(以及其他所有数据库)也一样
    OK

    redis> DBSIZE
    (integer) 0


.. _flushdb:

FLUSHDB
=========

**FLUSHDB**

清空\ *当前*\ 数据库中的所有 key 。

此命令从不失败。

**时间复杂度：**
    O(1)

**返回值：**
    总是返回 ``OK`` 。

::

    redis> DBSIZE    # 清空前的 key 数量
    (integer) 4

    redis> FLUSHDB
    OK

    redis> DBSIZE    # 清空后的 key 数量
    (integer) 0


.. _shutdown:

SHUTDOWN
=========

**SHUTDOWN**

`SHUTDOWN`_ 命令执行以下操作：

- 停止所有客户端
- 如果有最少一个保存点在等待，执行 `SAVE`_ 命令
- 如果 AOF 选项被打开，更新 AOF 文件
- 服务器关闭

如果持久化被打开的话， `SHUTDOWN`_ 命令会保证服务器正常关闭而\ *不*\ 丢失任何数据。

假如只是单纯地执行 `SAVE`_ 命令，然后再执行 :ref:`quit` 命令，则没有这一保证 —— 因为在执行 `SAVE`_ 之后、执行 :ref:`quit` 之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 :ref:`quit` 就会造成数据丢失。

**时间复杂度：**
    不明确

**返回值：**
    | 执行失败时返回错误。
    | 执行成功时不返回任何信息，服务器和客户端的连接断开，客户端自动退出。

::

    redis> PING
    PONG

    redis> SHUTDOWN  

    [huangz@mypad]$ 

    [huangz@mypad]$ redis
    Could not connect to Redis at: Connection refused
    not connected> 


.. _slowlog:

SLOWLOG
==========

.. function:: SLOWLOG subcommand [argument]

**什么是 SLOWLOG**

Slow log 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

**设置 SLOWLOG**

Slow log 的行为由两个配置参数(configuration parameter)指定，可以通过改写 redis.conf 文件或者用 ``CONFIG GET`` 和 ``CONFIG SET`` 命令对它们动态地进行修改。

第一个选项是 ``slowlog-log-slower-then`` ，它决定要对执行时间大于多少微秒(microsecond，1秒 = 1,000,000 微秒)的查询进行记录。

比如执行以下命令将让 slow log 记录所有查询时间大于等于 100 微秒的查询：

``CONFIG SET slowlog-log-slower-then 100`` ，

而以下命令记录所有查询时间大于 1000 微秒的查询：

``CONFIG SET slowlog-log-slower-then 1000`` 。

另一个选项是 ``slowlog-max-len`` ，它决定 slow log *最多*\ 能保存多少条日志， slow log 本身是一个 LIFO 队列，当队列大小超过 ``slowlog-max-len`` 时，最旧的一条日志将被删除，而最新的一条日志加入到 slow log ，以此类推。

以下命令让 slow log 最多保存 1000 条日志：

``CONFIG SET slowlog-max-len 1000`` 。

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
          3) "slowlog-log-slower-then"

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

**时间复杂度：**
    O(1)

**返回值：**
    取决于不同命令，返回不同的值。


.. _info:

INFO
======

**INFO**

返回关于 Redis 服务器的各种信息和统计值。

**时间复杂度：**
    O(1)

**返回值：**
    具体请参见下面的测试代码。

::

    redis> INFO
    redis_version:2.4.4             # Redis 的版本
    redis_git_sha1:00000000
    redis_git_dirty:0
    arch_bits:32
    multiplexing_api:epoll
    process_id:903                  # 当前 Redis 服务器进程id
    uptime_in_seconds:24612         # 运行时间(以秒计算)
    uptime_in_days:0                # 运行时间(以日计算)
    lru_clock:283730            
    used_cpu_sys:3.38
    used_cpu_user:2.15
    used_cpu_sys_children:0.11
    used_cpu_user_children:0.00
    connected_clients:1             # 连接的客户端数量
    connected_slaves:0              # 从属服务器的数量
    client_longest_output_list:0    
    client_biggest_input_buf:0
    blocked_clients:0
    used_memory:557304              # Redis 分配的内存总量
    used_memory_human:544.24K       
    used_memory_rss:17879040        # Redis 分配的内存总量(包括内存碎片)
    used_memory_peak:565904
    used_memory_peak_human:552.64K
    mem_fragmentation_ratio:32.08   # 内存碎片比率
    mem_allocator:jemalloc-2.2.5    # 目前使用的内存分配库
    loading:0   
    aof_enabled:0
    changes_since_last_save:2       # 上次保存数据库之后，执行命令的次数
    bgsave_in_progress:0            # 后台进行中的 save 操作的数量
    last_save_time:1324042687       # 最后一次成功保存的时间点，以 UNIX 时间戳格式显示
    bgrewriteaof_in_progress:0      # 后台进行中的 aof 文件修改操作的数量
    total_connections_received:16   # 运行以来连接过的客户端的总数量
    total_commands_processed:87     # 运行以来执行过的命令的总数量
    expired_keys:0                  # 运行以来过期的 key 的数量
    evicted_keys:0
    keyspace_hits:14                # 命中 key 的次数
    keyspace_misses:14              # 不命中 key 的次数
    pubsub_channels:0               # 当前使用中的频道数量
    pubsub_patterns:0               # 当前使用的模式的数量
    latest_fork_usec:314
    vm_enabled:0                    # 是否开启了 vm
    role:master
    db0:keys=6,expires=0            # 各个数据库的 key 的数量，以及带有生存期的 key 的数量
    db1:keys=6,expires=0
    db2:keys=1,expires=0


.. _config_get:

CONFIG GET
=============

.. function:: CONFIG GET parameter

`CONFIG GET`_ 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters)，不过并非所有配置参数都被 ``CONFIG GET`` 命令所支持。

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

**时间复杂度：**
    不明确

**返回值：**
    给定配置参数的值。

.. _config_set:

CONFIG SET
============

.. function:: CONFIG SET parameter value

`CONFIG SET`_ 命令可以动态地调整 Redis 服务器的配置(configuration)而无须重启。

你可以使用它修改配置参数，或者改变 Redis 的持久化(Persistence)方式。

`CONFIG SET`_ 可以修改的配置参数可以使用命令 ``CONFIG GET *`` 来列出，所有被 `CONFIG SET`_ 修改的配置参数都会立即生效。

关于 `CONFIG SET`_ 命令的更多消息，请参见命令 `CONFIG GET`_ 的说明。

关于如何使用 `CONFIG SET`_ 命令修改 Redis 持久化方式，请参见 `Redis Persistence <http://redis.io/topics/persistence>`_ 。

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


.. _config_resetstat:

CONFIG RESETSTAT
=================

.. function:: CONFIG RESETSTAT

重置 `INFO`_ 命令中的某些统计数据，包括：

- Keyspace hits (键空间命中次数)
- Keyspace misses (键空间不命中次数)
- Number of commands processed (执行命令的次数)
- Number of connections received (连接服务器的次数)
- Number of expired keys (过期key的数量)

**时间复杂度：**
    O(1)

**返回值：**
    总是返回 ``OK`` 。

::

    # 重置前的部分数据

    redis> INFO
    # ...
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:5
    # ...

    # 重置

    redis> CONFIG RESETSTAT
    OK

    # 重置后的部分数据

    redis> INFO
    # ...
    expired_keys:0
    evicted_keys:0
    keyspace_hits:0
    keyspace_misses:0
    pubsub_channels:0
    # ...


.. _debug_object:

DEBUG OBJECT
===============

.. function:: DEBUG OBJECT key

返回给定 ``key`` 的调试信息。

**时间复杂度：**
    O(1)

**返回值：**
    | 当 ``key`` 存在时，返回有关信息。
    | 当 ``key`` 不存在时，返回一个错误。 

::

    redis> DEBUG OBJECT my_pc
    Value at:0xb6838d20 refcount:1 encoding:raw serializedlength:9 lru:283790 lru_seconds_idle:150

    redis> DEBUG OBJECT your_mac
    (error) ERR no such key


.. _debug_segfault:

DEBUG SEGFAULT
===============

.. function:: DEBUG SEGFAULT

令 Redis 服务器崩溃，调试用。

**时间复杂度：**
    不明确

**返回值：**
    无

::

    redis> DEBUG SEGFAULT
    Could not connect to Redis at: Connection refused

    not connected> 


.. _monitor:

MONITOR
========

**MONITOR**

实时打印出 Redis 服务器接收到的命令，调试用。

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


.. _sync:

SYNC
=====

**SYNC**

用于复制功能(replication)的内部命令。

**时间复杂度：**
    不明确

**返回值：**
    不明确

::

    redis> SYNC
    "REDIS0002\xfe\x00\x00\auser_id\xc0\x03\x00\anumbers\xc2\xf3\xe0\x01\x00\x00\tdb_number\xc0\x00\x00\x04name\x06huangz\x00\anew_key\nhello_moto\x00\bgreeting\nhello moto\x00\x05my_pc\bthinkpad\x00\x04lock\xc0\x01\x00\nlock_times\xc0\x04\xfe\x01\t\x04info\x19\x02\x04name\b\x00zhangyue\x03age\x02\x0022\xff\t\aooredis,\x03\x04name\a\x00ooredis\aversion\x03\x001.0\x06author\x06\x00huangz\xff\x00\tdb_number\xc0\x01\x00\x05greet\x0bhello world\x02\nmy_friends\x02\x05marry\x04jack\x00\x04name\x05value\xfe\x02\x0c\x01s\x12\x12\x00\x00\x00\r\x00\x00\x00\x02\x00\x00\x01a\x03\xc0f'\xff\xff"
    (1.90s)
