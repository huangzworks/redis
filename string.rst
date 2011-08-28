.. _string_struct:

字符串(String)
***************

.. _set:

SET
===

.. function:: SET key value

将字符串值\ ``value``\ 关联到\ ``key``\ 。

如果\ ``key``\ 已经持有其他值，\ `SET`_\ 就覆写旧值，无视类型。

**时间复杂度：**
    O(1)

**返回值：**
    总是返回\ ``OK``\ ，因为\ `SET`_\ 不可能失败。

::

    # 情况1：对字符串类型的key进行SET

    redis> SET apple www.apple.com
    OK

    redis> GET apple
    "www.apple.com"


    # 情况2：对非字符串类型的key进行SET

    redis> LPUSH greet_list "hello"  # 建立一个列表
    (integer) 1

    redis> TYPE greet_list
    list

    redis> SET greet_list "yooooooooooooooooo"   # 覆盖列表类型
    OK

    redis> TYPE greet_list
    string

.. _setnx:

SETNX
=====

.. function:: SETNX key value

将\ ``key``\ 的值设为\ ``value``\ ，当且仅当\ ``key``\ 不存在。

若给定的\ ``key``\ 已经存在，则\ `SETNX`_\ 不做任何动作。

\ `SETNX`_\ 是"SET if Not eXists"(如果不存在，则SET)的简写。

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功，返回\ ``1``\ 。
    | 设置失败，返回\ ``0``\ 。

::
    
    redis> EXISTS job  # job不存在
    (integer) 0

    redis> SETNX job "programmer"  # job设置成功
    (integer) 1

    redis> SETNX job "code-farmer"  # job设置失败
    (integer) 0

    redis> GET job  # 没有被覆盖
    "programmer"

**设计模式(Design pattern): 将SETNX用于加锁(locking)**

\ `SETNX`_\ 可以用作加锁原语(locking primitive)。比如说，要对关键字(key)\ ``foo``\ 加锁，客户端可以尝试以下方式：

``SETNX lock.foo <current Unix time + lock timeout + 1>``

如果\ `SETNX`_\ 返回\ ``1``\ ，说明客户端已经获得了锁，\ ``key``\ 设置的unix时间则指定了锁失效的时间。之后客户端可以通过\ ``DEL lock.foo``\ 来释放锁。

如果\ `SETNX`_\ 返回\ ``0``\ ，说明\ ``key``\ 已经被其他客户端上锁了。如果锁是非阻塞(non blocking lock)的，我们可以选择返回调用，或者进入一个重试循环，直到成功获得锁或重试超时(timeout)。

**处理死锁(deadlock)**

上面的锁算法有一个问题：如果因为客户端失败、崩溃或其他原因导致没有办法释放锁的话，怎么办？

这种状况可以通过检测发现——因为上锁的\ ``key``\ 保存的是unix时间戳，假如\ ``key``\ 值的时间戳小于当前的时间戳，表示锁已经不再有效。  

但是，当有多个客户端同时检测一个锁是否过期并尝试释放它的时候，我们不能简单粗暴地删除死锁的\ ``key``\ ，再用\ `SETNX`_\ 上锁，因为这时竞争条件(race condition)已经形成了：

* C1和C2读取\ ``lock.foo``\ 并检查时间戳，\ `SETNX`_\ 都返回\ ``0``\ ，因为它已经被C3锁上了，但C3在上锁之后就崩溃(crashed)了。
* C1向\ ``lock.foo``\ 发送\ :ref:`del`\ 命令。
* C1向\ ``lock.foo``\ 发送\ `SETNX`_\ 并成功。
* C2向\ ``lock.foo``\ 发送\ :ref:`del`\ 命令。
* C2向\ ``lock.foo``\ 发送\ `SETNX`_\ 并成功。
* 出错：因为竞争条件的关系，C1和C2两个都获得了锁。

幸好，以下算法可以避免以上问题。来看看我们聪明的C4客户端怎么办：

* C4向\ ``lock.foo``\ 发送\ `SETNX`_\ 命令。
* 因为崩溃掉的C3还锁着\ ``lock.foo``\ ，所以Redis向C4返回\ ``0``\ 。
* C4向\ ``lock.foo``\ 发送\ `GET`_\ 命令，查看\ ``lock.foo``\ 的锁是否过期。如果不，则休眠(sleep)一段时间，并在之后重试。
* 另一方面，如果\ ``lock.foo``\ 内的unix时间戳比当前时间戳老，C4执行以下命令：

``GETSET lock.foo <current Unix timestamp + lock timeout + 1>``

* 因为\ `GETSET`_\ 的作用，C4可以检查看\ `GETSET`_\ 的返回值，确定\ ``lock.foo``\ 之前储存的旧值仍是那个过期时间戳，如果是的话，那么C4获得锁。
* 如果其他客户端，比如C5，比C4更快地执行了\ `GETSET`_\ 操作并获得锁，那么C4的\ `GETSET`_\ 操作返回的就是一个未过期的时间戳(C5设置的时间戳)。C4只好从第一步开始重试。

| 注意，即便C4的\ `GETSET`_\ 操作对\ ``key``\ 进行了修改，这对未来也没什么影响。
| (这里是不是有点问题？C4的确是可以重试，但C5怎么办？它的锁的过期被C4修改了。——译注)

.. warning:: 为了让这个加锁算法更健壮，获得锁的客户端应该常常检查过期时间以免锁因诸如\ :ref:`DEL`\ 等命令的执行而被意外解开，因为客户端失败的情况非常复杂，不仅仅是崩溃这么简单，还可能是客户端因为某些操作被阻塞了相当长时间，紧接着\ :ref:`DEL`\ 命令被尝试执行(但这时锁却在另外的客户端手上)。


.. _setex:

SETEX
======

.. function:: SETEX key seconds value 

将值\ ``value``\ 关联到\ ``key``\ ，并将\ ``key``\ 的生存时间设为\ ``seconds``\ (以秒为单位)。

如果\ ``key`` \ 已经存在，\ `SETEX`_\ 命令将覆写旧值。

这个命令类似于以下两个命令：

::

    SET key value
    EXPIRE key seconds  # 设置生存时间

不同之处是，\ `SETEX`_\ 是一个原子性(atomic)操作，关联值和设置生存时间两个动作会在同一时间内完成，该命令在Redis用作缓存时，非常实用。

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功时返回\ ``OK``\ 。
    | 当\ ``seconds``\ 参数不合法时，返回一个错误。

::

    # 情况1：key不存在

    redis> SETEX cache_user_id 60 10086
    OK

    redis> GET cache_user_id  # 值
    "10086"
     
     redis> TTL cache_user_id  # 剩余生存时间
     (integer) 49


    # 情况2：key已经存在，key被覆写

    redis> SET cd "timeless"
    OK

    redis> SETEX cd 3000 "goodbye my love"
    OK

    redis> GET cd
    "goodbye my love"


.. _setrange:

SETRANGE
=========

.. function:: SETRANGE key offset value

用\ ``value``\ 参数覆写(Overwrite)给定\ ``key``\ 所储存的字符串值，从偏移量\ ``offset``\ 开始。

不存在的\ ``key``\ 当作空白字符串处理。

\ `SETRANGE`_\ 命令会确保字符串足够长以便将\ ``value``\ 设置在指定的偏移量上，如果给定\ ``key``\ 原来储存的字符串长度比偏移量小(比如字符串只有\ ``5``\ 个字符长，但你设置的\ ``offset``\ 是\ ``10``\ )，那么原字符和偏移量之间的空白将用零比特(zerobytes,\ ``"\x00"``\ )来填充。

注意你能使用的最大偏移量是2^29-1(536870911)，因为Redis的字符串被限制在512兆(megabytes)内。如果你需要使用比这更大的空间，你得使用多个\ ``key``\ 。

**时间复杂度：**
    | 对小(small)的字符串，平摊复杂度O(1)。(关于什么字符串是"小"的，请参考\ `APPEND`_\ 命令)
    | 否则为O(M)，M为value参数的长度。

**返回值：**
    被\ `SETRANGE`_\ 修改之后，字符串的长度。

.. warning:: 
    当生成一个很长的字符串时，Redis需要分配内存空间，该操作有时候可能会造成服务器阻塞(block)。在2010年的Macbook Pro上，设置偏移量为536870911(512MB内存分配)，耗费约300毫秒，
    设置偏移量为134217728(128MB内存分配)，耗费约80毫秒，设置偏移量33554432(32MB内存分配)，耗费约30毫秒，设置偏移量为8388608(8MB内存分配)，耗费约8毫秒。
    注意若首次内存分配成功之后，再对同一个\ ``key``\ 调用\ `SETRANGE`_\ 操作，无须再重新内存。

**模式**

因为有了\ `SETRANGE`_\ 和\ `GETRANGE`_\ 命令，你可以将Redis字符串用作具有O(1)随机访问时间的线性数组。这在很多真实用例中都是非常快速且高效的储存方式。

::

    # 情况1：对非空字符串进行SETRANGE

    redis> SET greeting "hello world" 
    OK

    redis> SETRANGE greeting 6 "Redis"
    (integer) 11

    redis> GET greeting
    "hello Redis"


    # 情况2：对空字符串/不存在的key进行SETRANGE

    redis> EXISTS empty_string
    (integer) 0

    redis> SETRANGE empty_string 5 "Redis!"  # 对不存在的key使用SETRANGE
    (integer) 11

    redis> GET empty_string  # 空白处被"\x00"填充
    "\x00\x00\x00\x00\x00Redis!"


.. _mset:

MSET
=====

.. function:: MSET key value [key value ...]

同时设置一个或多个\ ``key-value``\ 对。

当发现同名的\ ``key``\ 存在时，\ `MSET`_\ 会用新值覆盖旧值，如果你不希望覆盖同名\ ``key``\ ，请使用\ `MSETNX`_\ 命令。  

\ `MSET`_\ 是一个原子性(atomic)操作，所有给定\ ``key``\ 都在同一时间内被设置，某些给定\ ``key``\ 被更新而另一些给定\ ``key``\ 没有改变的情况，不可能发生。

**时间复杂度：**
    O(N)，\ ``N``\ 为要设置的\ ``key``\ 数量。

**返回值：**
    总是返回\ ``OK``\ (因为\ ``MSET``\ 不可能失败)

::

    redis> MSET date "2011.4.18" time "9.09a.m." weather "sunny"    
    OK

    redis> KEYS *   # 确保指定的三个key-value对被插入
    1) "time"
    2) "weather"
    3) "date"

    redis> SET google "google.cn"  # MSET覆盖旧值的例子
    OK

    redis> MSET google "google.hk"
    OK

    redis> GET google
    "google.hk"


.. _msetnx:

MSETNX
========

.. function:: MSETNX key value [key value ...]

同时设置一个或多个\ ``key-value``\ 对，当且仅当\ ``key``\ 不存在。

即使\ *只有一个*\ \ ``key``\ 已存在，\ `MSETNX`_\ 也会拒绝\ *所有*\ 传入\ ``key``\ 的设置操作

`MSETNX`_\ 是原子性的，因此它可以用作设置多个不同\ ``key``\ 表示不同字段(field)的唯一性逻辑对象(unique logic object)，所有字段要么全被设置，要么全不被设置。

**时间复杂度：**
    O(N)，\ ``N``\ 为要设置的\ ``key``\ 的数量。

**返回值：**
    | 当所有\ ``key``\ 都成功设置，返回\ ``1``\ 。
    | 如果所有key都设置失败(最少有一个\ ``key``\ 已经存在)，那么返回\ ``0``\ 。

::

    # 情况1：对不存在的key进行MSETNX

    redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
    (integer) 1


    # 情况2：对已存在的key进行MSETNX

    redis> MSETNX rmdbs "Sqlite" language "python"  # rmdbs键已经存在，操作失败
    (integer) 0

    redis> EXISTS language  # 因为操作是原子性的，language没有被设置
    (integer) 0

    redis> GET rmdbs  # rmdbs没有被修改
    "MySQL"

    redis> MGET rmdbs nosql key-value-store  
    1) "MySQL"
    2) "MongoDB"
    3) "redis"


.. _append:

APPEND
======

.. function:: APPEND key value

如果\ ``key``\ 已经存在并且是一个字符串，\ `APPEND`_\ 命令将\ ``value``\ 追加到\ ``key``\ 原来的值之后。

如果\ ``key``\ 不存在，\ `APPEND`_\ 就简单地将给定\ ``key``\ 设为\ ``value``\ ，就像执行\ ``SET key value``\ 一样。

**时间复杂度：**
    平摊复杂度O(1)

**返回值：**
    追加\ ``value``\ 之后，\ ``key``\ 中字符串的长度。

::

    # 情况1：对不存在的key执行APPEND

    redis> EXISTS myphone  # 确保myphone不存在
    (integer) 0

    redis> APPEND myphone "nokia"  # 对不存在的key进行APPEND，等同于SET myphone "nokia"
    (integer) 5 # 字符长度


    # 情况2：对字符串进行APPEND

    redis> APPEND myphone " - 1110"  
    (integer) 12  # 长度从5个字符增加到12个字符

    redis> GET myphone  # 查看整个字符串
    "nokia - 1110"


.. _get:

GET
====

.. function:: GET key 
    
返回\ ``key``\ 所关联的字符串值。

如果\ ``key``\ 不存在则返回特殊值\ ``nil``\ 。

假如\ ``key``\ 储存的值不是字符串类型，返回一个错误，因为\ `GET`_\ 只能用于处理字符串值。

**时间复杂度：**
    O(1)

**返回值：**
    | \ ``key``\ 的值。
    | 如果\ ``key``\ 不存在，返回\ ``nil``\ 。

::

    redis> GET fake_key
    (nil)

    redis> SET animate "anohana"
    OK

    redis> GET animate
    "anohana"


.. _mget:

MGET
=====
.. function:: MGET key [key ...] 

返回所有(一个或多个)给定\ ``key``\ 的值。

如果某个指定\ ``key``\ 不存在，那么返回特殊值\ ``nil``\ 。因此，该命令永不失败。

**时间复杂度:**
    O(1)
                                        
**返回值：**
    一个包含所有给定\ ``key``\ 的值的列表。

::

    redis> MSET name huangz twitter twitter.com/huangz1990  #用MSET一次储存多个值
    OK

    redis> MGET name twitter
    1) "huangz"
    2) "twitter.com/huangz1990"

    redis> EXISTS fake_key
    (integer) 0

    redis> MGET name fake_key  # 当MGET中有不存在key的情况
    1) "huangz"
    2) (nil)


.. _getrange:

GETRANGE
=========

.. function:: GETRANGE key start end

返回\ ``key``\ 中字符串值的子字符串，字符串的截取范围由\ ``start``\ 和\ ``end``\ 两个偏移量决定(包括\ ``start``\ 和\ ``end``\ 在内)。

负数偏移量表示从字符串最后开始计数，\ ``-1``\ 表示最后一个字符，\ ``-2``\ 表示倒数第二个，以此类推。

\ `GETRANGE`_\ 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。

**时间复杂度：**
    | O(N)，\ ``N``\ 为要返回的字符串的长度。
    | 复杂度最终由返回值长度决定，但因为从已有字符串中建立子字符串的操作非常廉价(cheap)，所以对于长度不大的字符串，该操作的复杂度也可看作O(1)。

**返回值：**
    截取得出的子字符串。

.. note::
    在<=2.0的版本里，GETRANGE被叫作SUBSTR。

::

    redis> SET greeting "hello, my friend"
    OK

    redis> GETRANGE greeting 0 4  # 返回索引0-4的字符，包括4。
    "hello"

    redis> GETRANGE greeting -1 -5  # 不支持回绕操作
    ""

    redis> GETRANGE greeting -3 -1  # 负数索引
    "end"

    redis> GETRANGE greeting 0 -1  # 从第一个到最后一个
    "hello, my friend"

    redis> GETRANGE greeting 0 1008611  # 值域范围不超过实际字符串，超过部分自动被符略
    "hello, my friend"


.. _getset:

GETSET
========

.. function:: GETSET key value

将给定\ ``key``\ 的值设为\ ``value``\ ，并返回\ ``key``\ 的旧值。

当\ ``key``\ 存在但不是字符串类型时，返回一个错误。

**时间复杂度：**
    O(1)

**返回值：**
    | 返回给定\ ``key``\ 的旧值(old value)。
    | 当\ ``key``\ 没有旧值时，返回\ ``nil``\ 。

::

    redis> EXISTS mail 
    (integer) 0

    redis> GETSET mail xxx@google.com  # 因为mail之前不存在，没有旧值，返回nil
    (nil)

    redis> GETSET mail xxx@yahoo.com  # mail被更新，旧值被返回
    "xxx@google.com"

**设计模式**

\ `GETSET`_\ 可以和\ `INCR`_\ 组合使用，实现一个有原子性(atomic)复位操作的计数器(counter)。

举例来说，每次当某个事件发生时，进程可能对一个名为\ ``mycount``\ 的\ ``key``\ 调用\ `INCR`_\ 操作，通常我们还要在一个原子时间内同时完成获得计数器的值和将计数器值复位为\ ``0``\ 两个操作。

可以用命令\ ``GETSET mycounter 0``\ 来实现这一目标。

::
    
    redis> INCR mycount 
    (integer) 11

    redis> GETSET mycount 0  # 一个原子内完成GET mycount和SET mycount 0操作
    "11"

    redis> GET mycount
    "0"


.. _strlen:

STRLEN
=======

.. function:: STRLEN key

返回\ ``key``\ 所储存的字符串值的长度。

当\ ``key``\ 储存的不是字符串值时，返回一个错误。

复杂度：
    O(1)

返回值：
    | 字符串值的长度。
    | 当 \ ``key``\ 不存在时，返回\ ``0``\ 。

::

    redis> SET mykey "Hello world"
    OK

    redis> STRLEN mykey
    (integer) 11

    redis> STRLEN nonexisting # 不存在的key长度视为0
    (integer) 0


.. _decr:

DECR
=====

.. function:: DECR key

将\ ``key``\ 中储存的数字值减一。

如果\ ``key``\ 不存在，以\ ``0``\ 为\ ``key``\ 的初始值，然后执行\ `DECR`_\ 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在64位(bit)有符号数字表示之内。

关于更多递增(increment)/递减(decrement)操作信息，参见\ `INCR`_\ 命令。

**时间复杂度：**
    O(1)

**返回值：**
    执行\ `DECR`_\ 命令之后\ ``key``\ 的值。

::

    # 情况1：对存在的数字值key进行DECR

    redis> SET failure_times 10
    OK

    redis> DECR failure_times
    (integer) 9


    # 情况2：对不存在的key值进行DECR

    redis> EXISTS count 
    (integer) 0

    redis> DECR count
    (integer) -1


    # 情况3：对存在但不是数值的key进行DECR

    redis> SET company YOUR_CODE_SUCKS.LLC
    OK

    redis> DECR company
    (error) ERR value is not an integer or out of range


.. _decrby:

DECRBY
=======

.. function:: DECRBY key decrement

将\ ``key``\ 所储存的值减去减量\ ``decrement``\ 。

如果\ ``key``\ 不存在，以\ ``0``\ 为\ ``key``\ 的初始值，然后执行\ `DECRBY`_\ 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在64位(bit)有符号数字表示之内。

关于更多递增(increment)/递减(decrement)操作信息，参见\ `INCR`_\ 命令。

**时间复杂度：**
    O(1)

**返回值：**
    减去\ ``decrement``\ 之后，\ ``key``\ 的值。

::

    # 情况1：对存在的数值key进行DECRBY

    redis> SET count 100
    OK

    redis> DECRBY count 20
    (integer) 80

    
    # 情况2：对不存在的key进行DECRBY

    redis> EXISTS pages 
    (integer) 0

    redis> DECRBY pages 10  
    (integer) -10


.. _incr:

INCR
=====

.. function:: INCR key

将\ ``key``\ 中储存的数字值增一。

如果\ ``key``\ 不存在，以\ ``0``\ 为\ ``key``\ 的初始值，然后执行\ `INCR`_\ 操作。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在64位(bit)有符号数字表示之内。

**时间复杂度：**
    O(1)

**返回值：**
    执行\ `INCR`_\ 命令之后\ ``key``\ 的值。

.. note:: 
    这是一个针对字符串的操作，因为Redis没有专用的整数类型，所以key内储存的字符串被解释为十进制64位有符号整数来执行INCR操作。 

::
    
    redis> SET page_view 20
    OK

    redis> INCR page_view
    (integer) 21

    redis> GET page_view    # 数字值在Redis中以字符串的形式保存
    "21"


.. _incrby:

INCRBY
======

.. function:: INCRBY key increment

将\ ``key``\ 所储存的值加上增量\ ``increment``\ 。

如果\ ``key``\ 不存在，以\ ``0``\ 为\ ``key``\ 的初始值，然后执行\ `INCRBY`_\ 命令。

如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。

本操作的值限制在64位(bit)有符号数字表示之内。

关于更多递增(increment)/递减(decrement)操作信息，参见\ `INCR`_\ 命令。

**时间复杂度：**
    O(1)

**返回值：**
    加上\ ``increment``\ 之后，\ ``key``\ 的值。

::
    
    # 情况1：key存在且是数字值

    redis> SET rank 50  # 设置rank为50
    OK

    redis> INCRBY rank 20  # 给rank加上20
    (integer) 70

    redis> GET rank  
    "70"


    # 情况2：key不存在

    redis> EXISTS counter
    (integer) 0

    redis> INCRBY counter 30  
    (integer) 30

    redis> GET counter
    "30"


    # 情况3：key不是数字值

    redis> SET book "long long ago..."
    OK

    redis> INCRBY book 200
    (error) ERR value is not an integer or out of range


.. _setbit:

SETBIT
=======

.. function:: SETBIT key offset value 

对\ ``key``\ 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

位的设置或清除取决于\ ``value``\ 参数，可以是\ ``0``\ 也可以是\ ``1``\ 。

当\ ``key``\ 不存在时，自动生成一个新的字符串值。

字符串会增长(grown)以确保它可以将\ ``value``\ 保存在指定的偏移量上。当字符串值增长时，空白位置以\ ``0``\ 填充。

\ ``offset``\ 参数必须大于或等于\ ``0``\ ，小于2^32(bit映射被限制在512MB内)。


**时间复杂度:**
    O(1)

**返回值：**
    指定偏移量原来储存的位。

.. warning:: 对使用大的\ ``offset``\ 的\ `SETBIT`_\ 操作来说，内存分配可能造成Redis服务器被阻塞。具体参考\ `SETRANGE`_\ 命令，warning(警告)部分。

::

    redis> SETBIT bit 10086 1
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 1


.. _getbit:

GETBIT
======

.. function:: GETBIT key offset 

对\ ``key``\ 所储存的字符串值，获取指定偏移量上的位(bit)。

当\ ``offset``\ 比字符串值的长度大，或者\ ``key``\ 不存在时，返回\ ``0``\ 。
            
**时间复杂度：**
    O(1)

**返回值：**
    字符串值指定偏移量上的位(bit)。

::
    
    # 情况1：对不存在的key/不存在的offset进行GETBIT，
    #        默认为0

    redis> EXISTS bit
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 0

    
    # 情况2：对已存在的offset进行GETBIT

    redis> SETBIT bit 10086 1
    (integer) 0

    redis> GETBIT bit 10086
    (integer) 1
