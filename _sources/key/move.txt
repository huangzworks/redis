.. _move:

MOVE
====

**MOVE key db**

将当前数据库的 ``key`` 移动到给定的数据库 ``db`` 当中。

如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 ``key`` ，或者 ``key`` 不存在于当前数据库，那么 ``MOVE`` 没有任何效果。

因此，也可以利用这一特性，将 `MOVE`_ 当作锁(locking)原语(primitive)。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    移动成功返回 ``1`` ，失败则返回 ``0`` 。

::

    # key 存在于当前数据库

    redis> SELECT 0                             # redis默认使用数据库 0，为了清晰起见，这里再显式指定一次。
    OK

    redis> SET song "secret base - Zone"
    OK

    redis> MOVE song 1                          # 将 song 移动到数据库 1
    (integer) 1

    redis> EXISTS song                          # song 已经被移走
    (integer) 0

    redis> SELECT 1                             # 使用数据库 1
    OK

    redis:1> EXISTS song                        # 证实 song 被移到了数据库 1 (注意命令提示符变成了"redis:1"，表明正在使用数据库 1)
    (integer) 1
 

    # 当 key 不存在的时候 

    redis:1> EXISTS fake_key  
    (integer) 0

    redis:1> MOVE fake_key 0                    # 试图从数据库 1 移动一个不存在的 key 到数据库 0，失败
    (integer) 0

    redis:1> select 0                           # 使用数据库0
    OK

    redis> EXISTS fake_key                      # 证实 fake_key 不存在
    (integer) 0


    # 当源数据库和目标数据库有相同的 key 时

    redis> SELECT 0                             # 使用数据库0
    OK
    redis> SET favorite_fruit "banana"
    OK

    redis> SELECT 1                             # 使用数据库1
    OK
    redis:1> SET favorite_fruit "apple"
    OK

    redis:1> SELECT 0                           # 使用数据库0，并试图将 favorite_fruit 移动到数据库 1
    OK

    redis> MOVE favorite_fruit 1                # 因为两个数据库有相同的 key，MOVE 失败
    (integer) 0
    
    redis> GET favorite_fruit                   # 数据库 0 的 favorite_fruit 没变
    "banana"

    redis> SELECT 1
    OK

    redis:1> GET favorite_fruit                 # 数据库 1 的 favorite_fruit 也是
    "apple"
