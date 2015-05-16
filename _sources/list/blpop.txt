.. _blpop:

BLPOP
=======

**BLPOP key [key ...] timeout**

`BLPOP`_ 是列表的阻塞式(blocking)弹出原语。

它是 :ref:`LPOP` 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 `BLPOP`_ 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 ``key`` 参数时，按参数 ``key`` 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。

**非阻塞行为**

当 `BLPOP`_ 被调用时，如果给定 ``key`` 内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字一起，组成结果返回给调用者。

当存在多个给定 ``key`` 时， `BLPOP`_ 按给定 ``key`` 参数排列的先后顺序，依次检查各个列表。

假设现在有 ``job`` 、  ``command`` 和 ``request`` 三个列表，其中 ``job`` 不存在， ``command`` 和 ``request`` 都持有非空列表。考虑以下命令：

``BLPOP job command request 0``

`BLPOP`_ 保证返回的元素来自 ``command`` ，因为它是按"查找 ``job``  -> 查找 ``command``  -> 查找 ``request`` "这样的顺序，第一个找到的非空列表。

::

    redis> DEL job command request           # 确保key都被删除
    (integer) 0

    redis> LPUSH command "update system..."  # 为command列表增加一个值
    (integer) 1

    redis> LPUSH request "visit page"        # 为request列表增加一个值
    (integer) 1

    redis> BLPOP job command request 0       # job 列表为空，被跳过，紧接着 command 列表的第一个元素被弹出。
    1) "command"                             # 弹出元素所属的列表
    2) "update system..."                    # 弹出元素所属的值

**阻塞行为**

如果所有给定 ``key`` 都不存在或包含空列表，那么 `BLPOP`_ 命令将阻塞连接，直到等待超时，或有另一个客户端对给定 ``key`` 的任意一个执行 :ref:`LPUSH` 或 :ref:`RPUSH` 命令为止。

超时参数 ``timeout`` 接受一个以秒为单位的数字作为值。超时参数设为 ``0`` 表示阻塞时间可以无限期延长(block indefinitely) 。

::

    redis> EXISTS job                # 确保两个 key 都不存在
    (integer) 0
    redis> EXISTS command
    (integer) 0

    redis> BLPOP job command 300     # 因为key一开始不存在，所以操作会被阻塞，直到另一客户端对 job 或者 command 列表进行 PUSH 操作。
    1) "job"                         # 这里被 push 的是 job
    2) "do my home work"             # 被弹出的值
    (26.26s)                         # 等待的秒数

    redis> BLPOP job command 5       # 等待超时的情况
    (nil)
    (5.66s)                          # 等待的秒数

**相同的key被多个客户端同时阻塞**

相同的 ``key`` 可以被多个客户端同时阻塞。

不同的客户端被放进一个队列中，按『先阻塞先服务』(first-BLPOP，first-served)的顺序为 ``key`` 执行 `BLPOP`_ 命令。

**在MULTI/EXEC事务中的BLPOP**

`BLPOP`_ 可以用于流水线(pipline,批量地发送多个命令并读入多个回复)，但把它用在 :ref:`multi` / :ref:`exec` 块当中没有意义。因为这要求整个服务器被阻塞以保证块执行时的原子性，该行为阻止了其他客户端执行 :ref:`LPUSH` 或 :ref:`RPUSH` 命令。

因此，一个被包裹在 :ref:`multi` / :ref:`exec` 块内的 `BLPOP`_ 命令，行为表现得就像 :ref:`LPOP` 一样，对空列表返回 ``nil`` ，对非空列表弹出列表元素，不进行任何阻塞操作。

::

    # 对非空列表进行操作

    redis> RPUSH job programming
    (integer) 1

    redis> MULTI
    OK

    redis> BLPOP job 30
    QUEUED

    redis> EXEC           # 不阻塞，立即返回
    1) 1) "job"
       2) "programming"


    # 对空列表进行操作

    redis> LLEN job      # 空列表
    (integer) 0

    redis> MULTI
    OK

    redis> BLPOP job 30
    QUEUED

    redis> EXEC         # 不阻塞，立即返回
    1) (nil)

**可用版本：**  
    >= 2.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 如果列表为空，返回一个 ``nil`` 。
    | 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 ``key`` ，第二个元素是被弹出元素的值。

模式：事件提醒
------------------

有时候，为了等待一个新元素到达数据中，需要使用轮询的方式对数据进行探查。

另一种更好的方式是，使用系统提供的阻塞原语，在新元素到达时立即进行处理，而新元素还没到达时，就一直阻塞住，避免轮询占用资源。

对于 Redis ，我们似乎需要一个阻塞版的 :ref:`SPOP` 命令，但实际上，使用 `BLPOP`_ 或者 :ref:`BRPOP` 就能很好地解决这个问题。

使用元素的客户端(消费者)可以执行类似以下的代码：

::

    LOOP forever
        WHILE SPOP(key) returns elements
            ... process elements ...
        END
        BRPOP helper_key
    END

添加元素的客户端(生产者)则执行以下代码：

::

    MULTI
        SADD key element
        LPUSH helper_key x
    EXEC
