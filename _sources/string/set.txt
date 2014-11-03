.. _set:

SET
====

**SET key value [EX seconds] [PX milliseconds] [NX|XX]**

将字符串值 ``value`` 关联到 ``key`` 。

如果 ``key`` 已经持有其他值， `SET`_ 就覆写旧值，无视类型。

对于某个原本带有生存时间（TTL）的键来说，
当 :ref:`SET` 命令成功在这个键上执行时，
这个键原有的 TTL 将被清除。

**可选参数**

从 Redis 2.6.12 版本开始， :ref:`SET` 命令的行为可以通过一系列参数来修改：

- ``EX second`` ：设置键的过期时间为 ``second`` 秒。 ``SET key value EX second`` 效果等同于 ``SETEX key second value`` 。

- ``PX millisecond`` ：设置键的过期时间为 ``millisecond`` 毫秒。 ``SET key value PX millisecond`` 效果等同于 ``PSETEX key millisecond value`` 。

- ``NX`` ：只在键不存在时，才对键进行设置操作。 ``SET key value NX`` 效果等同于 ``SETNX key value`` 。

- ``XX`` ：只在键已经存在时，才对键进行设置操作。

.. note:: 因为 :ref:`SET` 命令可以通过参数来实现和 :ref:`SETNX` 、 :ref:`SETEX` 和 :ref:`PSETEX` 三个命令的效果，所以将来的 Redis 版本可能会废弃并最终移除 :ref:`SETNX` 、 :ref:`SETEX` 和 :ref:`PSETEX` 这三个命令。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    在 Redis 2.6.12 版本以前， :ref:`SET` 命令总是返回 ``OK`` 。
    
    | 从 Redis 2.6.12 版本开始， :ref:`SET` 在设置操作成功完成时，才返回 ``OK`` 。
    | 如果设置了 ``NX`` 或者 ``XX`` ，但因为条件没达到而造成设置操作未执行，那么命令返回空批量回复（NULL Bulk Reply）。


::

    # 对不存在的键进行设置

    redis 127.0.0.1:6379> SET key "value"
    OK

    redis 127.0.0.1:6379> GET key
    "value"


    # 对已存在的键进行设置

    redis 127.0.0.1:6379> SET key "new-value"
    OK

    redis 127.0.0.1:6379> GET key
    "new-value"

    
    # 使用 EX 选项

    redis 127.0.0.1:6379> SET key-with-expire-time "hello" EX 10086
    OK

    redis 127.0.0.1:6379> GET key-with-expire-time
    "hello"

    redis 127.0.0.1:6379> TTL key-with-expire-time
    (integer) 10069


    # 使用 PX 选项

    redis 127.0.0.1:6379> SET key-with-pexpire-time "moto" PX 123321
    OK

    redis 127.0.0.1:6379> GET key-with-pexpire-time
    "moto"

    redis 127.0.0.1:6379> PTTL key-with-pexpire-time
    (integer) 111939


    # 使用 NX 选项

    redis 127.0.0.1:6379> SET not-exists-key "value" NX   
    OK      # 键不存在，设置成功

    redis 127.0.0.1:6379> GET not-exists-key
    "value"

    redis 127.0.0.1:6379> SET not-exists-key "new-value" NX   
    (nil)   # 键已经存在，设置失败

    redis 127.0.0.1:6379> GEt not-exists-key
    "value" # 维持原值不变


    # 使用 XX 选项

    redis 127.0.0.1:6379> EXISTS exists-key
    (integer) 0

    redis 127.0.0.1:6379> SET exists-key "value" XX
    (nil)   # 因为键不存在，设置失败

    redis 127.0.0.1:6379> SET exists-key "value"
    OK      # 先给键设置一个值

    redis 127.0.0.1:6379> SET exists-key "new-value" XX
    OK      # 设置新值成功

    redis 127.0.0.1:6379> GET exists-key
    "new-value"


    # NX 或 XX 可以和 EX 或者 PX 组合使用

    redis 127.0.0.1:6379> SET key-with-expire-and-NX "hello" EX 10086 NX
    OK

    redis 127.0.0.1:6379> GET key-with-expire-and-NX
    "hello"

    redis 127.0.0.1:6379> TTL key-with-expire-and-NX
    (integer) 10063

    redis 127.0.0.1:6379> SET key-with-pexpire-and-XX "old value"
    OK

    redis 127.0.0.1:6379> SET key-with-pexpire-and-XX "new value" PX 123321
    OK

    redis 127.0.0.1:6379> GET key-with-pexpire-and-XX
    "new value"

    redis 127.0.0.1:6379> PTTL key-with-pexpire-and-XX
    (integer) 112999


    # EX 和 PX 可以同时出现，但后面给出的选项会覆盖前面给出的选项

    redis 127.0.0.1:6379> SET key "value" EX 1000 PX 5000000
    OK

    redis 127.0.0.1:6379> TTL key
    (integer) 4993  # 这是 PX 参数设置的值

    redis 127.0.0.1:6379> SET another-key "value" PX 5000000 EX 1000
    OK

    redis 127.0.0.1:6379> TTL another-key
    (integer) 997   # 这是 EX 参数设置的值




使用模式
---------------

命令 ``SET resource-name anystring NX EX max-lock-time`` 是一种在 Redis 中实现锁的简单方法。

客户端执行以上的命令：

- 如果服务器返回 ``OK`` ，那么这个客户端获得锁。

- 如果服务器返回 ``NIL`` ，那么客户端获取锁失败，可以在稍后再重试。

设置的过期时间到达之后，锁将自动释放。

可以通过以下修改，让这个锁实现更健壮：

- 不使用固定的字符串作为键的值，而是设置一个不可猜测（non-guessable）的长随机字符串，作为口令串（token）。

- 不使用 :ref:`DEL` 命令来释放锁，而是发送一个 Lua 脚本，这个脚本只在客户端传入的值和键的口令串相匹配时，才对键进行删除。

这两个改动可以防止持有过期锁的客户端误删现有锁的情况出现。

以下是一个简单的解锁脚本示例：

.. code-block:: lua

    if redis.call("get",KEYS[1]) == ARGV[1]
    then
        return redis.call("del",KEYS[1])
    else
        return 0
    end

这个脚本可以通过 ``EVAL ...script... 1 resource-name token-value`` 命令来调用。
