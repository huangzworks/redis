.. _list_struct:

表(List)
*********

**头元素和尾元素**

头元素指的是列表左端/前端第一个元素，尾元素指的是列表右端/后端第一个元素。

举个例子，列表\ ``list``\ 包含三个元素：\ ``x, y, z``\ ，其中\ ``x``\ 是头元素，而\ ``z``\ 则是尾元素。

**空列表**

指不包含任何元素的列表，Redis将不存在的\ ``key``\ 也视为空列表。


.. _lpush:

LPUSH
======

.. function:: LPUSH key value [value ...]

将一个或多个值\ ``value``\ 插入到列表\ ``key``\ 的\ *表头*\ 。

如果有多个\ ``value``\ 值，那么各个\ ``value``\ 值按从左到右的顺序依次插入到表头：比如对一个空列表(\ ``mylist``\ )执行\ ``LPUSH mylist a b c``\ ，则结果列表为\ ``c b a``\ ，等同于执行执行命令\ ``LPUSH mylist a``\ 、\ ``LPUSH mylist b``\ 、\ ``LPUSH mylist c``\ 。

如果\ ``key``\ 不存在，一个空列表会被创建并执行\ `LPUSH`_\ 操作。

当\ ``key``\ 存在但不是列表类型时，返回一个错误。

**时间复杂度：**
    O(1)

**返回值：**
    执行\ `LPUSH`_\ 命令后，列表的长度。

.. note:: 在Redis 2.4版本以前的\ `LPUSH`_\ 命令，都只接受单个\ ``value``\ 值。

::
    
    # 加入单个元素

    redis> LPUSH languages python
    (integer) 1

    # 加入重复元素

    redis> LPUSH languages python
    (integer) 2

    redis> LRANGE languages 0 -1 # 列表允许重复元素
    1) "python"
    2) "python"

    # 加入多个元素

    redis> LPUSH mylist a b c
    (integer) 3

    redis> LRANGE mylist 0 -1
    1) "c"
    2) "b"
    3) "a"

.. _lpushx:

LPUSHX
=======

.. function:: LPUSHX key value

将值\ ``value``\ 插入到列表\ ``key``\ 的表头，当且仅当\ ``key``\ 存在并且是一个列表。

和\ `LPUSH`_\ 命令相反，当\ ``key``\ 不存在时，\ `LPUSHX`_\ 命令什么也不做。
            
**时间复杂度：**
    O(1)

**返回值：**
    \ `LPUSHX`_\ 命令执行之后，表的长度。

::

    # 情况1：对空列表执行LPUSHX

    redis> LLEN greet    # greet是一个空列表
    (integer) 0

    redis> LPUSHX greet "hello"  # 尝试LPUSHX，失败，因为列表为空
    (integer) 0

    
    # 情况2：对非空列表执行LPUSHX

    redis> LPUSH greet "hello"   # 先用LPUSH创建一个有一个元素的列表
    (integer) 1

    redis> LPUSHX greet "good morning"   # 这次LPUSHX执行成功
    (integer) 2

    redis> LRANGE greet 0 -1
    1) "good morning"
    2) "hello"


.. _rpush:

RPUSH
========

.. function:: RPUSH key value [value ...]

将一个或多个值\ ``value``\ 插入到列表\ ``key``\ 的\ *表尾*\ 。

如果有多个\ ``value``\ 值，那么各个\ ``value``\ 值按从左到右的顺序依次插入到表尾：比如对一个空列表(\ ``mylist``\ )执行\ ``RPUSH mylist a b c``\ ，则结果列表为\ ``a b c``\ ，等同于执行命令\ ``RPUSH mylist a``\ 、\ ``RPUSH mylist b``\ 、\ ``RPUSH mylist c``\ 。

如果\ ``key``\ 不存在，一个空列表会被创建并执行\ `RPUSH`_\ 操作。

当\ ``key``\ 存在但不是列表类型时，返回一个错误。

**时间复杂度：**
    O(1)

**返回值：**
    执行\ `RPUSH`_\ 操作后，表的长度。

.. note:: 在Redis 2.4版本以前的\ `RPUSH`_\ 命令，都只接受单个\ ``value``\ 值。

::

    # 添加单个元素

    redis> RPUSH languages c
    (integer) 1

    # 添加重复元素

    redis> RPUSH languages c
    (integer) 2

    redis> LRANGE languages 0 -1 # 列表允许重复元素
    1) "c"
    2) "c"

    # 添加多个元素

    redis> RPUSH mylist a b c
    (integer) 3

    redis> LRANGE mylist 0 -1
    1) "a"
    2) "b"
    3) "c"


.. _rpushx:

RPUSHX
=======

.. function:: RPUSHX key value 

将值\ ``value``\ 插入到列表\ ``key``\ 的表尾，当且仅当\ ``key``\ 存在并且是一个列表。

和\ `RPUSH`_\ 命令相反，当\ ``key``\ 不存在时，\ `RPUSHX`_\ 命令什么也不做。
            
**时间复杂度：**
    O(1)

**返回值：**
    \ `RPUSHX`_\ 命令执行之后，表的长度。

::

    # 情况1：key不存在

    redis> LLEN greet
    (integer) 0

    redis> RPUSHX greet "hello"  # 对不存在的key进行RPUSHX，PUSH失败。
    (integer) 0

    
    # 情况2：key存在且是一个非空列表

    redis> RPUSH greet "hi"  # 先用RPUSH插入一个元素
    (integer) 1

    redis> RPUSHX greet "hello"  # greet现在是一个列表类型，RPUSHX操作成功。
    (integer) 2

    redis> LRANGE greet 0 -1
    1) "hi"
    2) "hello"


.. _lpop:

LPOP
=======

.. function:: LPOP key

移除并返回列表\ ``key``\ 的头元素。 

**时间复杂度：**
    O(1)

**返回值：**
    | 列表的头元素。
    | 当\ ``key``\ 不存在时，返回\ ``nil``\ 。

::

    redis> LLEN course
    (integer) 0

    redis> RPUSH course algorithm001
    (integer) 1
    redis> RPUSH course c++101
    (integer) 2

    redis> LPOP course  # 移除头元素
    "algorithm001"


.. _rpop:

RPOP
=======

.. function:: RPOP key

移除并返回列表\ ``key``\ 的尾元素。 

**时间复杂度：**
    O(1)

**返回值：**
    | 列表的尾元素。
    | 当\ ``key``\ 不存在时，返回\ ``nil``\ 。

::

    redis> RPUSH mylist "one"
    (integer) 1
    redis> RPUSH mylist "two"
    (integer) 2
    redis> RPUSH mylist "three"
    (integer) 3

    redis> RPOP mylist  # 返回被弹出的元素
    "three"

    redis> LRANGE mylist 0 -1   # 列表剩下的元素 
    1) "one"
    2) "two"


.. _blpop:

BLPOP
=======

.. function:: BLPOP key [key ...] timeout 

\ `BLPOP`_\ 是列表的阻塞式(blocking)弹出原语。

它是\ `LPOP`_\ 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被\ `BLPOP`_\ 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个\ ``key``\ 参数时，按参数\ ``key``\ 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。

**非阻塞行为**

当\ `BLPOP`_\ 被调用时，如果给定\ ``key``\ 内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字一起，组成结果返回给调用者。

当存在多个给定\ ``key``\ 时，\ `BLPOP`_\ 按给定\ ``key``\ 参数排列的先后顺序，依次检查各个列表。

假设现在有\ ``job``\ 、 \ ``command``\ 和\ ``request``\ 三个列表，其中\ ``job``\ 不存在，\ ``command``\ 和\ ``request``\ 都持有非空列表。考虑以下命令：

``BLPOP job command request 0``

\ `BLPOP`_\ 保证返回的元素来自\ ``command``\ ，因为它是按"查找\ ``job``\  -> 查找\ ``command``\  -> 查找\ ``request``\ "这样的顺序，第一个找到的非空列表。

::

    redis> DEL job command request  # 确保key都被删除
    (integer) 0
    redis> LPUSH command "update system..."  # 为command列表增加一个值
    (integer) 1
    redis> LPUSH request "visit page"  # 为request列表增加一个值
    (integer) 1

    redis> BLPOP job command request 0  # job列表为空，被跳过，紧接着command列表的第一个元素被弹出。
    1) "command"    # 弹出元素所属的列表
    2) "update system..."   # 弹出元素所属的值

**阻塞行为**

如果所有给定\ ``key``\ 都不存在或包含空列表，那么\ `BLPOP`_\ 命令将阻塞连接，直到等待超时，或有另一个客户端对给定\ ``key``\ 的任意一个执行\ `LPUSH`_\ 或\ `RPUSH`_\ 命令为止。

超时参数\ ``timeout``\ 接受一个以秒为单位的数字作为值。超时参数设为\ ``0``\ 表示阻塞时间可以无限期延长(block indefinitely) 。

::

    redis> EXISTS job  # 确保两个key都不存在
    (integer) 0
    redis> EXISTS command
    (integer) 0

    redis> BLPOP job command 300  #因为key一开始不存在，所以操作会被阻塞，直到另一客户端对job或者command列表进行PUSH操作。
    1) "job"  # 这里被push的是job
    2) "do my home work"  # 被弹出的值
    (26.26s)  # 等待的秒数

    redis> BLPOP job command 5  # 等待超时的情况
    (nil)
    (5.66s) # 等待的秒数

**相同的key被多个客户端同时阻塞**

| 相同的\ ``key``\ 可以被多个客户端同时阻塞。
| 不同的客户端被放进一个队列中，按"先阻塞先服务"(first-BLPOP，first-served)的顺序为\ ``key``\ 执行\ `BLPOP`_\ 命令。

**在MULTI/EXEC事务中的BLPOP**

\ `BLPOP`_\ 可以用于流水线(pipline,批量地发送多个命令并读入多个回复)，但把它用在\ :ref:`multi`\ /\ :ref:`exec`\ 块当中没有意义。因为这要求整个服务器被阻塞以保证块执行时的原子性，该行为阻止了其他客户端执行\ `LPUSH`_\ 或\ `RPUSH`_\ 命令。

因此，一个被包裹在\ :ref:`multi`\ /\ :ref:`exec`\ 块内的\ `BLPOP`_\ 命令，行为表现得就像\ `LPOP`_\ 一样，对空列表返回\ ``nil``\ ，对非空列表弹出列表元素，不进行任何阻塞操作。

::

    # 情况1：对非空列表进行操作

    redis> RPUSH job programming
    (integer) 1

    redis> MULTI
    OK

    redis> BLPOP job 30
    QUEUED

    redis> EXEC  # 不阻塞，立即返回
    1) 1) "job"
       2) "programming"


    # 情况2：对空列表进行操作

    redis> LLEN job  # 空列表
    (integer) 0

    redis> MULTI
    OK

    redis> BLPOP job 30
    QUEUED

    redis> EXEC  # 不阻塞，立即返回
    1) (nil)

**时间复杂度：**
    O(1)

**返回值：**
    | 如果列表为空，返回一个\ ``nil``\ 。
    | 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的\ ``key``\ ，第二个元素是被弹出元素的值。


.. _brpop:

BRPOP
=======

.. function:: BRPOP key [key ...] timeout

\ `BRPOP`_\ 是列表的阻塞式(blocking)弹出原语。

它是\ `RPOP`_\ 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被\ `BRPOP`_\ 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个\ ``key``\ 参数时，按参数\ ``key``\ 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。

关于阻塞操作的更多信息，请查看\ `BLPOP`_\ 命令，\ `BRPOP`_\ 除了弹出元素的位置和\ `BLPOP`_\ 不同之外，其他表现一致。

**时间复杂度：**
    O(1)

**返回值：**
    | 假如在指定时间内没有任何元素被弹出，则返回一个\ ``nil``\ 和等待时长。
    | 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的\ ``key``\ ，第二个元素是被弹出元素的值。

::

    redis> LLEN course
    (integer) 0

    redis> RPUSH course algorithm001
    (integer) 1
    redis> RPUSH course c++101  # 尾部元素
    (integer) 2

    redis> BRPOP course 30
    1) "course" # 弹出元素的key
    2) "c++101" # 弹出元素的值


.. _llen:

LLEN
=======

.. function:: LLEN key

返回列表\ ``key``\ 的长度。

如果\ ``key``\ 不存在，则\ ``key``\ 被解释为一个空列表，返回\ ``0``\ .

如果\ ``key``\ 不是列表类型，返回一个错误。 

**时间复杂度：**
    O(1)

**返回值：**
    列表\ ``key``\ 的长度。

::
    
    # 情况1：空列表

    redis> LLEN job 
    (integer) 0


    # 情况2：非空列表

    redis> LPUSH job "cook food"
    (integer) 1
    redis> LPUSH job "have lunch"
    (integer) 2

    redis> LLEN job
    (integer) 2


.. _lrange:

LRANGE
=======

.. function:: LRANGE key start stop

返回列表\ ``key``\ 中指定区间内的元素，区间以偏移量\ ``start``\ 和\ ``stop``\ 指定。

下标(index)参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 表示列表的第一个元素，以\ ``1``\ 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以\ ``-1``\ 表示列表的最后一个元素，\ ``-2``\ 表示列表的倒数第二个元素，以此类推。

**注意LRANGE命令和编程语言区间函数的区别**

假如你有一个包含一百个元素的列表，对该列表执行\ ``LRANGE list 0 10``\ ，结果是一个包含11个元素的列表，这表明\ ``stop``\ 下标也在\ `LRANGE`_\ 命令的取值范围之内(闭区间)，这和某些语言的区间函数可能不一致，比如Ruby的\ ``Range.new``\ 、\ ``Array#slice``\ 和Python的\ ``range()``\ 函数。

**超出范围的下标**

超出范围的下标值不会引起错误。

如果\ ``start``\ 下标比列表的最大下标\ ``end``\ (\ ``LLEN list``\ 减去\ ``1``\ )还要大，或者\ ``start > stop``\ ，\ `LRANGE`_\ 返回一个空列表。

如果\ ``stop``\ 下标比\ ``end``\ 下标还要大，Redis将\ ``stop``\ 的值设置为\ ``end``\ 。


**时间复杂度:**
    O(S+N)，\ ``S``\ 为偏移量\ ``start``\ ，\ ``N``\ 为指定区间内元素的数量。

**返回值:**
    一个列表，包含指定区间内的元素。

::

    redis> RPUSH fp-language lisp   # 插入一个值到列表fp-language
    (integer) 1
    redis> LRANGE fp-language 0 0 
    1) "lisp"

    redis> RPUSH fp-language scheme
    (integer) 2
    redis> LRANGE fp-language 0 1
    1) "lisp"
    2) "scheme"


.. _lrem:

LREM
=======

.. function:: LREM key count value 

根据参数\ ``count``\ 的值，移除列表中与参数\ ``value``\ 相等的元素。
        
\ ``count``\ 的值可以是以下几种：
    * \ ``count > 0``\ : 从表头开始向表尾搜索，移除与\ ``value``\ 相等的元素，数量为\ ``count``\ 。
    * \ ``count < 0``\ : 从表尾开始向表头搜索，移除与\ ``value``\ 相等的元素，数量为\ ``count``\ 的绝对值。
    * \ ``count = 0``\ : 移除表中所有与\ ``value``\ 相等的值。

**时间复杂度：**
    O(N)，\ ``N``\ 为列表的长度。

**返回值：**
    | 被移除元素的数量。
    | 因为不存在的\ ``key``\ 被视作空表(empty list)，所以当\ ``key``\ 不存在时，\ `LREM`_\ 命令总是返回\ ``0``\ 。

:: 

    # 先创建一个表，内容排列是
    # morning hello morning helllo morning

    redis> LPUSH greet "morning"
    (integer) 1
    redis> LPUSH greet "hello"
    (integer) 2
    redis> LPUSH greet "morning"
    (integer) 3
    redis> LPUSH greet "hello"
    (integer) 4
    redis> LPUSH greet "morning"
    (integer) 5

    redis> LRANGE greet 0 4 # 查看所有元素
    1) "morning"
    2) "hello"
    3) "morning"
    4) "hello"
    5) "morning"

    redis> LREM greet 2 morning  # 移除从表头到表尾，最先发现的两个morning
    (integer) 2  # 两个元素被移除

    redis> LLEN greet   # 还剩3个元素
    (integer) 3

    redis> LRANGE greet 0 2
    1) "hello"
    2) "hello"
    3) "morning"

    redis> LREM greet -1 morning  # 移除从表尾到表头，第一个morning
    (integer) 1

    redis> LLEN greet
    (integer) 2

    redis> LRANGE greet 0 1
    1) "hello"
    2) "hello"

    redis> LREM greet 0 hello  # 移除表中所有hello
    (integer) 2  # 两个hello被移除

    redis> LLEN greet
    (integer) 0


.. _lset:

LSET
=======

.. function:: LSET key index value 

将列表\ ``key``\ 下标为\ ``index``\ 的元素的值甚至为\ ``value``\ 。

更多信息请参考\ `LINDEX`_\ 操作。 

当\ ``index``\ 参数超出范围，或对一个空列表(\ ``key``\ 不存在)进行\ `LSET`_\ 时，返回一个错误。

**时间复杂度：**
    | 对头元素或尾元素进行\ `LSET`_\ 操作，复杂度为O(1)。
    | 其他情况下，为O(N)，\ ``N``\ 为列表的长度。

**返回值：**
    操作成功返回\ ``ok``\ ，否则返回错误信息。

::

    # 情况1：对空列表(key不存在)进行LSET

    redis> EXISTS list
    (integer) 0

    redis> LSET list 0 item
    (error) ERR no such key


    # 情况2：对非空列表进行LSET

    redis> LPUSH job "cook food"
    (integer) 1

    redis> LRANGE job 0 0
    1) "cook food"

    redis> LSET job 0 "play game"
    OK

    redis> LRANGE job  0 0
    1) "play game"


    # 情况3：index超出范围

    redis> LLEN list # 列表长度为1
    (integer) 1

    redis> LSET list 3 'out of range'
    (error) ERR index out of range


.. _ltrim:

LTRIM
=======

.. function:: LTRIM key start stop

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

举个例子，执行命令\ ``LTRIM list 0 2``\ ，表示只保留列表\ ``list``\ 的前三个元素，其余元素全部删除。

下标(index)参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 表示列表的第一个元素，以\ ``1``\ 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以\ ``-1``\ 表示列表的最后一个元素，\ ``-2``\ 表示列表的倒数第二个元素，以此类推。

当\ ``key``\ 不是列表类型时，返回一个错误。

\ `LTRIM`_\ 命令通常和\ `LPUSH`_\ 命令或\ `RPUSH`_\ 命令配合使用，举个例子：

::

    LPUSH log newest_log
    LTRIM log 0 99

这个例子模拟了一个日志程序，每次将最新日志\ ``newest_log``\ 放到\ ``log``\ 列表中，并且只保留最新的\ ``100``\ 项。注意当这样使用\ ``LTRIM``\ 命令时，时间复杂度是O(1)，因为平均情况下，每次只有一个元素被移除。

**注意LTRIM命令和编程语言区间函数的区别**

假如你有一个包含一百个元素的列表\ ``list``\ ，对该列表执行\ ``LTRIM list 0 10``\ ，结果是一个包含11个元素的列表，这表明\ ``stop``\ 下标也在\ `LTRIM`_\ 命令的取值范围之内(闭区间)，这和某些语言的区间函数可能不一致，比如Ruby的\ ``Range.new``\ 、\ ``Array#slice``\ 和Python的\ ``range()``\ 函数。

**超出范围的下标**

超出范围的下标值不会引起错误。

如果\ ``start``\ 下标比列表的最大下标\ ``end``\ (\ ``LLEN list``\ 减去\ ``1``\ )还要大，或者\ ``start > stop``\ ，\ `LTRIM`_\ 返回一个空列表(因为\ `LTRIM`_\ 已经将整个列表清空)。

如果\ ``stop``\ 下标比\ ``end``\ 下标还要大，Redis将\ ``stop``\ 的值设置为\ ``end``\ 。

**时间复杂度:**
    O(N)，\ ``N``\ 为被移除的元素的数量。

**返回值:**
    | 命令执行成功时，返回\ ``ok``\ 。

::

    # 情况1：一般情况下标

    redis> LRANGE alpha 0 -1 # 建立一个5元素的列表
    1) "h"
    2) "e"
    3) "l"
    4) "l"
    5) "o"

    redis> LTRIM alpha 1 -1  # 删除索引为0的元素
    OK

    redis> LRANGE alpha 0 -1 # "h"被删除
    1) "e"
    2) "l"
    3) "l"
    4) "o"

    
    # 情况2：stop下标比元素的最大下标要大

    redis> LTRIM alpha 1 10086 
    OK
    redis> LRANGE alpha 0 -1
    1) "l"
    2) "l"
    3) "o"

    
    # 情况3：start和stop下标都比最大下标要大，且start < sotp

    redis> LTRIM alpha 10086 200000  
    OK
    redis> LRANGE alpha 0 -1 # 整个列表被清空，等同于DEL alpha
    (empty list or set)


    # 情况4：start > stop

    redis> LRANGE alpha 0 -1 # 在新建一个列表
    1) "h"
    2) "u"
    3) "a"
    4) "n"
    5) "g"
    6) "z"

    redis> LTRIM alpha 10086 4
    OK

    redis> LRANGE alpha 0 -1 # 列表同样被清空
    (empty list or set)


.. _lindex:

LINDEX
=======

.. function:: LINDEX key index

返回列表\ ``key``\ 中，下标为\ ``index``\ 的元素。

下标(index)参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 表示列表的第一个元素，以\ ``1``\ 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以\ ``-1``\ 表示列表的最后一个元素，\ ``-2``\ 表示列表的倒数第二个元素，以此类推。

如果\ ``key``\ 不是列表类型，返回一个错误。

**时间复杂度：**
    | O(N)，\ ``N``\ 为到达下标\ ``index``\ 过程中经过的元素数量。
    | 因此，对列表的头元素和尾元素执行\ `LINDEX`_\ 命令，复杂度为O(1)。

**返回值:**
    | 列表中下标为\ ``index``\ 的元素。
    | 如果\ ``index``\ 参数的值不在列表的区间范围内(out of range)，返回\ ``nil``\ 。

::

    redis> LPUSH mylist "World"
    (integer) 1

    redis> LPUSH mylist "Hello"
    (integer) 2

    redis> LINDEX mylist 0
    "Hello"

    redis> LINDEX mylist -1
    "World"

    redis> LINDEX mylist 3  # index不在mylist的区间范围内
    (nil)


.. _linsert:

LINSERT
=========

.. function:: LINSERT key BEFORE|AFTER pivot value

将值\ ``value``\ 插入到列表\ ``key``\ 当中，位于值\ ``pivot``\ 之前或之后。

当\ ``pivot``\ 不存在于列表\ ``key``\ 时，不执行任何操作。

当\ ``key``\ 不存在时，\ ``key``\ 被视为空列表，不执行任何操作。

如果\ ``key``\ 不是列表类型，返回一个错误。 

**时间复杂度:**
    O(N)，\ ``N``\ 为寻找\ ``pivot``\ 过程中经过的元素数量。

**返回值:**
    | 如果命令执行成功，返回插入操作完成之后，列表的长度。
    | 如果没有找到\ ``pivot``\ ，返回\ ``-1``\ 。
    | 如果\ ``key``\ 不存在或为空列表，返回\ ``0``\ 。

::

    redis> RPUSH mylist "Hello"
    (integer) 1
    redis> RPUSH mylist "World"
    (integer) 2

    redis> LINSERT mylist BEFORE "World" "There"
    (integer) 3

    redis> LRANGE mylist 0 -1
    1) "Hello"
    2) "There"
    3) "World"

    redis> LINSERT mylist BEFORE "go" "let's"    # 对一个非空列表插入，查找一个不存在的pivot
    (integer) -1    # 失败

    redis> EXISTS fake_list  # 对一个空列表执行LINSERT命令
    (integer) 0

    redis> LINSERT fake_list BEFORE "nono" "gogogog"
    (integer) 0 # 失败


.. _rpoplpush:

RPOPLPUSH
===========

.. function:: RPOPLPUSH source destination

命令\ `RPOPLPUSH`_\ 在一个原子时间内，执行以下两个动作：

    - 将列表\ ``source``\ 中的最后一个元素(尾元素)弹出，并返回给客户端。
    - 将\ ``source``\ 弹出的元素插入到列表\ ``destination``\ ，作为\ ``destination``\ 列表的的头元素。

举个例子，你有两个列表\ ``source``\ 和\ ``destination``\ ，\ ``source``\ 列表有元素\ ``a, b, c``\ ，\ ``destination``\ 列表有元素\ ``x, y, z``\ ，执行\ ``RPOPLPUSH source destination``\ 之后，\ ``source``\ 列表包含元素\ ``a, b``\ ，\ ``destination``\ 列表包含元素\ ``c, x, y, z`` \ ，并且元素\ ``c``\ 被返回。

如果\ ``source``\ 不存在，值\ ``nil``\ 被返回，并且不执行其他动作。

如果\ ``source``\ 和\ ``destination``\ 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转(rotation)操作。

**时间复杂度：**
    O(1)

**返回值：**
    被弹出的元素。

::

    # 相关数据

    redis> RPUSH alpha a
    (integer) 1
    redis> RPUSH alpha b
    (integer) 2
    redis> RPUSH alpha c
    (integer) 3
    redis> RPUSH alpha d
    (integer) 4

    # 情况1：source和destination不同

    redis> LRANGE alpha 0 -1 # 查看所有元素
    1) "a"
    2) "b"
    3) "c"
    4) "d"

    redis> RPOPLPUSH alpha reciver   # 执行一次RPOPLPUSH看看
    "d"

    redis> LRANGE alpha 0 -1 
    1) "a"
    2) "b"
    3) "c"

    redis> LRANGE reciver 0 -1
    1) "d"

    redis> RPOPLPUSH alpha reciver   # 再执行一次，确保rpop和lpush的位置正确
    "c"

    redis> LRANGE alpha 0 -1
    1) "a"
    2) "b"

    redis> LRANGE reciver 0 -1
    1) "c"
    2) "d"

    
    # 情况2：source和destination相同

    redis> RPOPLPUSH alpha alpha 
    "c"

    redis> LRANGE alpha 0 -1 # 原来的尾元素"c"被放到了头部
    1) "c"
    2) "a"
    3) "b"

**设计模式： 一个安全的队列**

Redis的列表经常被用作队列(queue)，用于在不同程序之间有序地交换消息(message)。一个程序(称之为生产者，producer)通过\ `LPUSH`_\ 命令将消息放入队列中，而另一个程序(称之为消费者，consumer)通过\ `RPOP`_\ 命令取出队列中等待时间最长的消息。

不幸的是，在这个过程中，一个消费者可能在获得一个消息之后崩溃，而未执行完成的消息也因此丢失。

使用\ `RPOPLPUSH`_\ 命令可以解决这个问题，因为它在返回一个消息之余，还将该消息添加到另一个列表当中，另外的这个列表可以用作消息的备份表：假如一切正常，当消费者完成该消息的处理之后，可以用\ `LREM`_\ 命令将该消息从备份表删除。

另一方面，助手(helper)程序可以通过监视备份表，将超过一定处理时限的消息重新放入队列中去(负责处理该消息的消费者可能已经崩溃)，这样就不会丢失任何消息了。


.. _brpoplpush:

BRPOPLPUSH
===========

.. function:: BRPOPLPUSH source destination timeout

\ `BRPOPLPUSH`_\ 是\ `RPOPLPUSH`_\ 的阻塞版本，当给定列表\ ``source``\ 不为空时，\ `BRPOPLPUSH`_\ 的表现和\ `RPOPLPUSH`_\ 一样。

当列表\ ``source``\ 为空时，\ `BRPOPLPUSH`_\ 命令将阻塞连接，直到等待超时，或有另一个客户端对\ ``source``\ 执行\ `LPUSH`_\ 或\ `RPUSH`_\ 命令为止。

超时参数\ ``timeout``\ 接受一个以秒为单位的数字作为值。超时参数设为\ ``0``\ 表示阻塞时间可以无限期延长(block indefinitely) 。

更多相关信息，请参考\ `RPOPLPUSH`_\ 命令。

**时间复杂度：**
    O(1)

**返回值：**
    | 假如在指定时间内没有任何元素被弹出，则返回一个\ ``nil``\ 和等待时长。
    | 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。

::

    # 情况1：非空列表

    redis> BRPOPLPUSH msg reciver 500
    "hello moto"    # 弹出元素的值
    (3.38s)         # 等待时长

    redis> LLEN reciver
    (integer) 1

    redis> LRANGE reciver 0 0
    1) "hello moto"


    # 情况2：空列表

    redis> BRPOPLPUSH msg reciver 1 
    (nil)
    (1.34s)
