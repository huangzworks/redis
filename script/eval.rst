.. _eval:

EVAL
======

**EVAL script numkeys key [key ...] arg [arg ...]**

从 Redis 2.6.0 版本开始，通过内置的 Lua 解释器，可以使用 `EVAL`_ 命令对 Lua 脚本进行求值。

``script`` 参数是一段 Lua 5.1 脚本程序，它会被运行在 Redis 服务器上下文中，这段脚本不必(也不应该)定义为一个 Lua 函数。

``numkeys`` 参数用于指定键名参数的个数。

键名参数 ``key [key ...]`` 从 `EVAL`_ 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 ``KEYS`` 数组，用 ``1`` 为基址的形式访问( ``KEYS[1]`` ， ``KEYS[2]`` ，以此类推)。

在命令的最后，那些不是键名参数的附加参数 ``arg [arg ...]`` ，可以在 Lua 中通过全局变量 ``ARGV`` 数组访问，访问的形式和 ``KEYS`` 变量类似( ``ARGV[1]`` 、 ``ARGV[2]`` ，诸如此类)。

上面这几段长长的说明可以用一个简单的例子来概括：

::

    > eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
    1) "key1"
    2) "key2"
    3) "first"
    4) "second"

其中 ``"return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}"`` 是被求值的 Lua 脚本，数字 ``2`` 指定了键名参数的数量， ``key1`` 和 ``key2`` 是键名参数，分别使用 ``KEYS[1]`` 和 ``KEYS[2]`` 访问，而最后的 ``first`` 和 ``second`` 则是附加参数，可以通过 ``ARGV[1]`` 和 ``ARGV[2]`` 访问它们。

在 Lua 脚本中，可以使用两个不同函数来执行 Redis 命令，它们分别是：

- ``redis.call()``
- ``redis.pcall()``

这两个函数的唯一区别在于它们使用不同的方式处理执行命令所产生的错误，在后面的『错误处理』部分会讲到这一点。

``redis.call()`` 和 ``redis.pcall()`` 两个函数的参数可以是任何格式良好(well formed)的 Redis 命令：

::

    > eval "return redis.call('set','foo','bar')" 0
    OK

需要注意的是，上面这段脚本的确实现了将键 ``foo`` 的值设为 ``bar`` 的目的，但是，它违反了  `EVAL`_ 命令的语义，因为脚本里使用的所有键都应该由 ``KEYS`` 数组来传递，就像这样：

::

    > eval "return redis.call('set',KEYS[1],'bar')" 1 foo
    OK

要求使用正确的形式来传递键(key)是有原因的，因为不仅仅是 `EVAL`_ 这个命令，所有的 Redis 命令，在执行之前都会被分析，籍此来确定命令会对哪些键进行操作。

因此，对于 `EVAL`_ 命令来说，必须使用正确的形式来传递键，才能确保分析工作正确地执行。除此之外，使用正确的形式来传递键还有很多其他好处，它的一个特别重要的用途就是确保 Redis 集群可以将你的请求发送到正确的集群节点。(对 Redis 集群的工作还在进行当中，但是脚本功能被设计成可以与集群功能保持兼容。)不过，这条规矩并不是强制性的，从而使得用户有机会滥用(abuse) Redis 单实例配置(single instance configuration)，代价是这样写出的脚本不能被 Redis 集群所兼容。


在 Lua 数据类型和 Redis 数据类型之间转换
------------------------------------------------

当 Lua 通过 ``call()`` 或 ``pcall()`` 函数执行 Redis 命令的时候，命令的返回值会被转换成 Lua 数据结构。同样地，当 Lua 脚本在 Redis 内置的解释器里运行时，Lua 脚本的返回值也会被转换成 Redis 协议(protocol)，然后由 `EVAL`_ 将值返回给客户端。

数据类型之间的转换遵循这样一个设计原则：如果将一个 Redis 值转换成 Lua 值，之后再将转换所得的 Lua 值转换回 Redis 值，那么这个转换所得的 Redis 值应该和最初时的 Redis 值一样。

换句话说， Lua 类型和 Redis 类型之间存在着一一对应的转换关系。

以下列出的是详细的转换规则：

从 Redis 转换到 Lua ：

- Redis integer reply -> Lua number / Redis 整数转换成 Lua 数字
- Redis bulk reply -> Lua string / Redis bulk 回复转换成 Lua 字符串
- Redis multi bulk reply -> Lua table (may have other Redis data types nested) / Redis 多条 bulk 回复转换成 Lua 表，表内可能有其他别的 Redis 数据类型
- Redis status reply -> Lua table with a single ok field containing the status / Redis 状态回复转换成 Lua 表，表内的 ``ok`` 域包含了状态信息
- Redis error reply -> Lua table with a single err field containing the error / Redis 错误回复转换成 Lua 表，表内的 ``err`` 域包含了错误信息
- Redis Nil bulk reply and Nil multi bulk reply -> Lua false boolean type / Redis 的 Nil 回复和 Nil 多条回复转换成 Lua 的布尔值 ``false``

从 Lua 转换到 Redis：

- Lua number -> Redis integer reply / Lua 数字转换成 Redis 整数
- Lua string -> Redis bulk reply / Lua 字符串转换成 Redis bulk 回复
- Lua table (array) -> Redis multi bulk reply / Lua 表(数组)转换成 Redis 多条 bulk 回复
- Lua table with a single ok field -> Redis status reply / 一个带单个 ``ok`` 域的 Lua 表，转换成 Redis 状态回复
- Lua table with a single err field -> Redis error reply / 一个带单个 ``err`` 域的 Lua 表，转换成 Redis 错误回复
- Lua boolean false -> Redis Nil bulk reply / Lua 的布尔值 ``false`` 转换成 Redis 的 Nil bulk 回复

从 Lua 转换到 Redis 有一条额外的规则，这条规则没有和它对应的从 Redis 转换到 Lua 的规则：

- Lua boolean true -> Redis integer reply with value of 1 / Lua 布尔值 ``true`` 转换成 Redis 整数回复中的 ``1`` 

以下是几个类型转换的例子：

::

    > eval "return 10" 0
    (integer) 10

    > eval "return {1,2,{3,'Hello World!'}}" 0
    1) (integer) 1
    2) (integer) 2
    3) 1) (integer) 3
       2) "Hello World!"

    > eval "return redis.call('get','foo')" 0
    "bar"

在上面的三个代码示例里，前两个演示了如何将 Lua 值转换成 Redis 值，最后一个例子更复杂一些，它演示了一个将 Redis 值转换成 Lua 值，然后再将 Lua 值转换成 Redis 值的类型转过程。


脚本的原子性
------------------

Redis 使用单个 Lua 解释器去运行所有脚本，并且， Redis 也保证脚本会以原子性(atomic)的方式执行：当某个脚本正在运行的时候，不会有其他脚本或 Redis 命令被执行。这和使用 :ref:`MULTI` / :ref:`EXEC` 包围的事务很类似。在其他别的客户端看来，脚本的效果(effect)要么是不可见的(not visible)，要么就是已完成的(already completed)。

另一方面，这也意味着，执行一个运行缓慢的脚本并不是一个好主意。写一个跑得很快很顺溜的脚本并不难，因为脚本的运行开销(overhead)非常少，但是当你不得不使用一些跑得比较慢的脚本时，请小心，因为当这些蜗牛脚本在慢吞吞地运行的时候，其他客户端会因为服务器正忙而无法执行命令。


错误处理
------------

前面的命令介绍部分说过， ``redis.call()`` 和 ``redis.pcall()`` 的唯一区别在于它们对错误处理的不同。

当 ``redis.call()`` 在执行命令的过程中发生错误时，脚本会停止执行，并返回一个脚本错误，错误的输出信息会说明错误造成的原因：

::

    redis> lpush foo a
    (integer) 1

    redis> eval "return redis.call('get', 'foo')" 0
    (error) ERR Error running script (call to f_282297a0228f48cd3fc6a55de6316f31422f5d17): ERR Operation against a key holding the wrong kind of value 

和 ``redis.call()`` 不同， ``redis.pcall()`` 出错时并不引发(raise)错误，而是返回一个带 ``err`` 域的 Lua 表(table)，用于表示错误：

::

    redis 127.0.0.1:6379> EVAL "return redis.pcall('get', 'foo')" 0
    (error) ERR Operation against a key holding the wrong kind of value

带宽和 EVALSHA
-------------------

`EVAL`_ 命令要求你在每次执行脚本的时候都发送一次脚本主体(script body)。Redis 有一个内部的缓存机制，因此它不会每次都重新编译脚本，不过在很多场合，付出无谓的带宽来传送脚本主体并不是最佳选择。

为了减少带宽的消耗， Redis 实现了 EVALSHA 命令，它的作用和 `EVAL`_ 一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的 SHA1 校验和(sum)。

EVALSHA 命令的表现如下：

- 如果服务器还记得给定的 SHA1 校验和所指定的脚本，那么执行这个脚本
- 如果服务器不记得给定的 SHA1 校验和所指定的脚本，那么它返回一个特殊的错误，提醒用户使用 `EVAL`_ 代替 EVALSHA

以下是示例：

::

    > set foo bar
    OK

    > eval "return redis.call('get','foo')" 0
    "bar"

    > evalsha 6b1bf486c81ceb7edf3c093f4c48582e38c0e791 0
    "bar"

    > evalsha ffffffffffffffffffffffffffffffffffffffff 0
    (error) `NOSCRIPT` No matching script. Please use [EVAL](/commands/eval).

客户端库的底层实现可以一直乐观地使用 EVALSHA 来代替 `EVAL`_ ，并期望着要使用的脚本已经保存在服务器上了，只有当 ``NOSCRIPT`` 错误发生时，才使用 `EVAL`_ 命令重新发送脚本，这样就可以最大限度地节省带宽。

这也说明了执行 `EVAL`_ 命令时，使用正确的格式来传递键名参数和附加参数的重要性：因为如果将参数硬写在脚本中，那么每次当参数改变的时候，都要重新发送脚本，即使脚本的主体并没有改变，相反，通过使用正确的格式来传递键名参数和附加参数，就可以在脚本主体不变的情况下，直接使用 EVALSHA 命令对脚本进行复用，免去了无谓的带宽消耗。


脚本缓存
--------------

Redis 保证所有被运行过的脚本都会被永久保存在脚本缓存当中，这意味着，当 `EVAL`_ 命令在一个 Redis 实例上成功执行某个脚本之后，随后针对这个脚本的所有 EVALSHA 命令都会成功执行。

刷新脚本缓存的唯一办法是显式地调用 ``SCRIPT FLUSH`` 命令，这个命令会清空运行过的所有脚本的缓存。通常只有在云计算环境中，Redis 实例被改作其他客户或者别的应用程序的实例时，才会执行这个命令。

缓存可以长时间储存而不产生内存问题的原因是，它们的体积非常小，而且数量也非常少，即使脚本在概念上类似于实现一个新命令，即使在一个大规模的程序里有成百上千的脚本，即使这些脚本会经常修改，即便如此，储存这些脚本的内存仍然是微不足道的。

事实上，用户会发现 Redis 不移除缓存中的脚本实际上是一个好主意。比如说，对于一个和 Redis 保持持久化链接(persistent connection)的程序来说，它可以确信，执行过一次的脚本会一直保留在内存当中，因此它可以在流水线中使用 EVALSHA 命令而不必担心因为找不到所需的脚本而产生错误(稍候我们会看到在流水线中执行脚本的相关问题)。


SCRIPT 命令
-----------------

Redis 提供了以下几个 SCRIPT 命令，用于对脚本子系统(scripting subsystem)进行控制：

- :ref:`script_flush` ：清除所有脚本缓存
- :ref:`script_exists` ：根据给定的脚本校验和，检查指定的脚本是否存在于脚本缓存
- :ref:`script_load` ：将一个脚本装入脚本缓存，但并不立即运行它
- :ref:`script_kill` ：杀死当前正在运行的脚本


纯函数脚本
----------------

在编写脚本方面，一个重要的要求就是，脚本应该被写成纯函数(pure function)。

也就是说，脚本应该具有以下属性：

- 对于同样的数据集输入，给定相同的参数，脚本执行的 Redis 写命令总是相同的。脚本执行的操作不能依赖于任何隐藏(非显式)数据，不能依赖于脚本在执行过程中、或脚本在不同执行时期之间可能变更的状态，并且它也不能依赖于任何来自 I/O 设备的外部输入。

使用系统时间(system time)，调用像 :ref:`RANDOMKEY` 那样的随机命令，或者使用 Lua 的随机数生成器，类似以上的这些操作，都会造成脚本的求值无法每次都得出同样的结果。

为了确保脚本符合上面所说的属性， Redis 做了以下工作：

- Lua 没有访问系统时间或者其他内部状态的命令

- Redis 会返回一个错误，阻止这样的脚本运行： 这些脚本在执行随机命令之后(比如 :ref:`RANDOMKEY` 、 :ref:`SRANDMEMBER` 或 :ref:`TIME` 等)，还会执行可以修改数据集的 Redis 命令。如果脚本只是执行只读操作，那么就没有这一限制。注意，随机命令并不一定就指那些带 RAND 字眼的命令，任何带有非确定性的命令都会被认为是随机命令，比如 :ref:`TIME` 命令就是这方面的一个很好的例子。

- 每当从 Lua 脚本中调用那些返回无序元素的命令时，执行命令所得的数据在返回给 Lua 之前会先执行一个静默(slient)的字典序排序(`lexicographical sorting <http://en.wikipedia.org/wiki/Lexicographical_order>`_)。举个例子，因为 Redis 的 Set 保存的是无序的元素，所以在 Redis 命令行客户端中直接执行 :ref:`SMEMBERS` ，返回的元素是无序的，但是，假如在脚本中执行 ``redis.call("smembers", KEYS[1])`` ，那么返回的总是排过序的元素。

- 对 Lua 的伪随机数生成函数 ``math.random`` 和 ``math.randomseed`` 进行修改，使得每次在运行新脚本的时候，总是拥有同样的 seed 值。这意味着，每次运行脚本时，只要不使用 ``math.randomseed`` ，那么 ``math.random`` 产生的随机数序列总是相同的。

尽管有那么多的限制，但用户还是可以用一个简单的技巧写出带随机行为的脚本(如果他们需要的话)。

假设现在我们要编写一个 Redis 脚本，这个脚本从列表中弹出 N 个随机数。一个 Ruby 写的例子如下：

::

    require 'rubygems'
    require 'redis'

    r = Redis.new

    RandomPushScript = <<EOF
        local i = tonumber(ARGV[1])
        local res
        while (i > 0) do
            res = redis.call('lpush',KEYS[1],math.random())
            i = i-1
        end
        return res
    EOF

    r.del(:mylist)
    puts r.eval(RandomPushScript,[:mylist],[10,rand(2**32)])

这个程序每次运行都会生成带有以下元素的列表：

::

    > lrange mylist 0 -1
    1) "0.74509509873814"
    2) "0.87390407681181"
    3) "0.36876626981831"
    4) "0.6921941534114"
    5) "0.7857992587545"
    6) "0.57730350670279"
    7) "0.87046522734243"
    8) "0.09637165539729"
    9) "0.74990198051087"
    10) "0.17082803611217"

上面的 Ruby 程序每次都只生成同样的列表，用途并不是太大。那么，该怎样修改这个脚本，使得它仍然是一个纯函数(符合 Redis 的要求)，但是每次调用都可以产生不同的随机元素呢？

一个简单的办法是，为脚本添加一个额外的参数，让这个参数作为 Lua 的随机数生成器的 seed 值，这样的话，只要给脚本传入不同的 seed ，脚本就会生成不同的列表元素。

以下是修改后的脚本：

::

    RandomPushScript = <<EOF
        local i = tonumber(ARGV[1])
        local res
        math.randomseed(tonumber(ARGV[2]))
        while (i > 0) do
            res = redis.call('lpush',KEYS[1],math.random())
            i = i-1
        end
        return res
    EOF

    r.del(:mylist)
    puts r.eval(RandomPushScript,1,:mylist,10,rand(2**32))

尽管对于同样的 seed ，上面的脚本产生的列表元素是一样的(因为它是一个纯函数)，但是只要每次在执行脚本的时候传入不同的 seed ，我们就可以得到带有不同随机元素的列表。

Seed 会在复制(replication link)和写 AOF 文件时作为一个参数来传播，保证在载入 AOF 文件或附属节点(slave)处理脚本时， seed 仍然可以及时得到更新。

注意，Redis 实现保证 ``math.random`` 和 ``math.randomseed`` 的输出和运行 Redis 的系统架构无关，无论是 32 位还是 64 位系统，无论是小端(little endian)还是大端(big endian)系统，这两个函数的输出总是相同的。


全局变量保护
---------------

为了防止不必要的数据泄漏进 Lua 环境， Redis 脚本不允许创建全局变量。如果一个脚本需要在多次执行之间维持某种状态，它应该使用 Redis key 来进行状态保存。

企图在脚本中访问一个全局变量(不论这个变量是否存在)将引起脚本停止， `EVAL`_ 命令会返回一个错误：

::

    redis 127.0.0.1:6379> eval 'a=10' 0
    (error) ERR Error running script (call to f_933044db579a2f8fd45d8065f04a8d0249383e57): user_script:1: Script attempted to create global variable 'a'

Lua 的 debug 工具，或者其他设施，比如打印（alter）用于实现全局保护的 meta table ，都可以用于实现全局变量保护。

实现全局变量保护并不难，不过有时候还是会不小心而为之。一旦用户在脚本中混入了 Lua 全局状态，那么 AOF 持久化和复制（replication）都会无法保证，所以，请不要使用全局变量。

避免引入全局变量的一个诀窍是：将脚本中用到的所有变量都使用 ``local`` 关键字定义为局部变量。
    

库
-------

Redis 内置的 Lua 解释器加载了以下 Lua 库：

- ``base``
- ``table``
- ``string``
- ``math``
- ``debug``
- ``cjson``
- ``cmsgpack``

其中 ``cjson`` 库可以让 Lua 以非常快的速度处理 JSON 数据，除此之外，其他别的都是 Lua 的标准库。

每个 Redis 实例都保证会加载上面列举的库，从而确保每个 Redis 脚本的运行环境都是相同的。


使用脚本散发 Redis 日志
-------------------------------

在 Lua 脚本中，可以通过调用 ``redis.log`` 函数来写 Redis 日志(log)：

``redis.log(loglevel, message)``

其中， ``message`` 参数是一个字符串，而 ``loglevel`` 参数可以是以下任意一个值：

- ``redis.LOG_DEBUG``
- ``redis.LOG_VERBOSE``
- ``redis.LOG_NOTICE``
- ``redis.LOG_WARNING``

上面的这些等级(level)和标准 Redis 日志的等级相对应。

对于脚本散发(emit)的日志，只有那些和当前 Redis 实例所设置的日志等级相同或更高级的日志才会被散发。

以下是一个日志示例：

``redis.log(redis.LOG_WARNING, "Something is wrong with this script.")``

执行上面的函数会产生这样的信息：

``[32343] 22 Mar 15:21:39 # Something is wrong with this script.``


沙箱(sandbox)和最大执行时间
----------------------------------

脚本应该仅仅用于传递参数和对 Redis 数据进行处理，它不应该尝试去访问外部系统(比如文件系统)，或者执行任何系统调用。

除此之外，脚本还有一个最大执行时间限制，它的默认值是 5 秒钟，一般正常运作的脚本通常可以在几分之几毫秒之内完成，花不了那么多时间，这个限制主要是为了防止因编程错误而造成的无限循环而设置的。

最大执行时间的长短由 ``lua-time-limit`` 选项来控制(以毫秒为单位)，可以通过编辑 ``redis.conf`` 文件或者使用 :ref:`config_get` 和 :ref:`config_set` 命令来修改它。

当一个脚本达到最大执行时间的时候，它并不会自动被 Redis 结束，因为 Redis 必须保证脚本执行的原子性，而中途停止脚本的运行意味着可能会留下未处理完的数据在数据集(data set)里面。

因此，当脚本运行的时间超过最大执行时间后，以下动作会被执行：

- Redis 记录一个脚本正在超时运行
- Redis 开始重新接受其他客户端的命令请求，但是只有 ``SCRIPT KILL`` 和 ``SHUTDOWN NOSAVE`` 两个命令会被处理，对于其他命令请求， Redis 服务器只是简单地返回 ``BUSY`` 错误。
- 可以使用 ``SCRIPT KILL`` 命令将一个仅执行只读命令的脚本杀死，因为只读命令并不修改数据，因此杀死这个脚本并不破坏数据的完整性
- 如果脚本已经执行过写命令，那么唯一允许执行的操作就是 ``SHUTDOWN NOSAVE`` ，它通过停止服务器来阻止当前数据集写入磁盘


流水线(pipeline)上下文(context)中的 EVALSHA
----------------------------------------------

在流水线请求的上下文中使用 EVALSHA 命令时，要特别小心，因为在流水线中，必须保证命令的执行顺序。

一旦在流水线中因为 EVALSHA 命令而发生 NOSCRIPT 错误，那么这个流水线就再也没有办法重新执行了，否则的话，命令的执行顺序就会被打乱。

为了防止出现以上所说的问题，客户端库实现应该实施以下的其中一项措施：

- 总是在流水线中使用 `EVAL`_ 命令
- 检查流水线中要用到的所有命令，找到其中的 `EVAL`_ 命令，并使用 :ref:`SCRIPT_EXISTS` 命令检查要用到的脚本是不是全都已经保存在缓存里面了。如果所需的全部脚本都可以在缓存里找到，那么就可以放心地将所有 `EVAL`_ 命令改成 EVALSHA 命令，否则的话，就要在流水线的顶端(top)将缺少的脚本用 :ref:`script_load` 命令加上去。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    `EVAL`_ 和 EVALSHA 可以在 O(1) 复杂度内找到要被执行的脚本，其余的复杂度取决于执行的脚本本身。
