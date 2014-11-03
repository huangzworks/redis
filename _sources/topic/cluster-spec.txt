.. _cluster_spec:

Redis 集群规范
===========================

.. note::

    本文档翻译自 http://redis.io/topics/cluster-spec 。

引言
--------------------------------

这个文档是正在开发中的 Redis 集群功能的规范（specification）文档，
文档分为两个部分：

- 第一部分介绍目前已经在 ``unstable`` 分支中实现了的那些功能。

- 第二部分介绍目前仍未实现的那些功能。

文档各个部分的内容可能会随着集群功能的设计修改而发生改变，
其中，
未实现功能发生修改的几率比已实现功能发生修改的几率要高。

这个规范包含了编写客户端库（client library）所需的全部知识，
不过请注意，
这里列出的一部分细节可能会在未来发生变化。


什么是 Redis 集群？
--------------------------------

Redis 集群是一个分布式（distributed）、容错（fault-tolerant）的 Redis 实现，
集群可以使用的功能是普通单机 Redis 所能使用的功能的一个子集（subset）。

Redis 集群中不存在中心（central）节点或者代理（proxy）节点，
集群的其中一个主要设计目标是达到线性可扩展性（linear scalability）。

Redis 集群为了保证一致性（consistency）而牺牲了一部分容错性：
系统会在保证对网络断线（net split）和节点失效（node failure）具有有限（limited）抵抗力的前提下，
尽可能地保持数据的一致性。

.. note::

    集群将节点失效视为网络断线的其中一种特殊情况。

集群的容错功能是通过使用主节点（master）和从节点（slave）两种角色（role）的节点（node）来实现的：

- 主节点和从节点使用完全相同的服务器实现，
  它们的功能（functionally）也完全一样，
  但从节点通常仅用于替换失效的主节点。

- 不过，
  如果不需要保证“先写入，后读取”操作的一致性（read-after-write consistency），
  那么可以使用从节点来执行只读查询。


Redis 集群实现的功能子集
--------------------------------

Redis 集群实现了单机 Redis 中，
所有处理单个数据库键的命令。

针对多个数据库键的复杂计算操作，
比如集合的并集操作、合集操作没有被实现，
那些理论上需要使用多个节点的多个数据库键才能完成的命令也没有被实现。

在将来，
用户也许可以通过 :ref:`MIGRATE COPY <MIGRATE>` 命令，
在集群的计算节点（computation node）中执行针对多个数据库键的只读操作，
但集群本身不会去实现那些需要将多个数据库键在多个节点中移来移去的复杂多键命令。

Redis 集群不像单机 Redis 那样支持多数据库功能，
集群只使用默认的 ``0`` 号数据库，
并且不能使用 :ref:`SELECT` 命令。


Redis 集群协议中的客户端和服务器
-----------------------------------------------------------

Redis 集群中的节点有以下责任：

- 持有键值对数据。

- 记录集群的状态，包括键到正确节点的映射（mapping keys to right nodes）。

- 自动发现其他节点，识别工作不正常的节点，并在有需要时，在从节点中选举出新的主节点。

为了执行以上列出的任务，
集群中的每个节点都与其他节点建立起了“集群连接（cluster bus）”，
该连接是一个 TCP 连接，
使用二进制协议进行通讯。

节点之间使用 `Gossip 协议 <http://en.wikipedia.org/wiki/Gossip_protocol>`_ 来进行以下工作：

- 传播（propagate）关于集群的信息，以此来发现新的节点。

- 向其他节点发送 ``PING`` 数据包，以此来检查目标节点是否正常运作。

- 在特定事件发生时，发送集群信息。

除此之外，
集群连接还用于在集群中发布或订阅信息。

因为集群节点不能代理（proxy）命令请求，
所以客户端应该在节点返回 ``-MOVED`` 或者 ``-ASK`` 转向（redirection）错误时，
自行将命令请求转发至其他节点。

因为客户端可以自由地向集群中的任何一个节点发送命令请求，
并可以在有需要时，
根据转向错误所提供的信息，
将命令转发至正确的节点，
所以在理论上来说，
客户端是无须保存集群状态信息的。

不过，
如果客户端可以将键和节点之间的映射信息保存起来，
可以有效地减少可能出现的转向次数，
籍此提升命令执行的效率。


键分布模型
-----------------------------------------------------------

Redis 集群的键空间被分割为 ``16384`` 个槽（slot），
集群的最大节点数量也是 ``16384`` 个。

.. note::

    推荐的最大节点数量为 1000 个左右。

每个主节点都负责处理 ``16384`` 个哈希槽的其中一部分。

当我们说一个集群处于“稳定”（stable）状态时，
指的是集群没有在执行重配置（reconfiguration）操作，
每个哈希槽都只由一个节点进行处理。

.. note::
    
    重配置指的是将某个/某些槽从一个节点移动到另一个节点。

.. note::

    一个主节点可以有任意多个从节点，
    这些从节点用于在主节点发生网络断线或者节点失效时，
    对主节点进行替换。

以下是负责将键映射到槽的算法：

::

    HASH_SLOT = CRC16(key) mod 16384

以下是该算法所使用的参数：

- 算法的名称: XMODEM (又称 ZMODEM 或者 CRC-16/ACORN)

- 结果的长度: 16 位

- 多项数（poly）: 1021 (也即是 ``x16 + x12 + x5 + 1``\ )

- 初始化值: ``0000``

- 反射输入字节（Reflect Input byte）: ``False``

- 发射输出 CRC （Reflect Output CRC）: ``False``

- 用于 CRC 输出值的异或常量（Xor constant to output CRC）: ``0000``

- 该算法对于输入 ``"123456789"`` 的输出: ``31C3``

附录 A 中给出了集群所使用的 CRC16 算法的实现。

CRC16 算法所产生的 16 位输出中的 14 位会被用到。

在我们的测试中，
CRC16 算法可以很好地将各种不同类型的键平稳地分布到 ``16384`` 个槽里面。


集群节点属性
-----------------------------------------------------------

每个节点在集群中都有一个独一无二的 ID ，
该 ID 是一个十六进制表示的 160 位随机数，
在节点第一次启动时由 ``/dev/urandom`` 生成。

节点会将它的 ID 保存到配置文件，
只要这个配置文件不被删除，
节点就会一直沿用这个 ID 。

节点 ID 用于标识集群中的每个节点。
一个节点可以改变它的 IP 和端口号，
而不改变节点 ID 。
集群可以自动识别出 IP/端口号的变化，
并将这一信息通过 Gossip 协议广播给其他节点知道。

.. 
    The cluster is also able to detect the change in IP/port 
    and reconfigure broadcast the information using the gossip protocol running over the cluster bus.

以下是每个节点都有的关联信息，
并且节点会将这些信息发送给其他节点：

- 节点所使用的 IP 地址和 TCP 端口号。

- 节点的标志（flags）。

- 节点负责处理的哈希槽。

- 节点最近一次使用集群连接发送 ``PING`` 数据包（packet）的时间。

- 节点最近一次在回复中接收到 ``PONG`` 数据包的时间。

- 集群将该节点标记为下线的时间。

- 该节点的从节点数量。

- 如果该节点是从节点的话，那么它会记录主节点的节点 ID 。
  如果这是一个主节点的话，那么主节点 ID 这一栏的值为 ``0000000`` 。

以上信息的其中一部分可以通过向集群中的任意节点（主节点或者从节点都可以）发送 ``CLUSTER NODES`` 命令来获得。

以下是一个向集群中的主节点发送 ``CLUSTER NODES`` 命令的例子，
该集群由三个节点组成：

::

    $ redis-cli cluster nodes
    d1861060fe6a534d42d8a19aeb36600e18785e04 :0 myself - 0 1318428930 connected 0-1364
    3886e65cc906bfd9b1f7e7bde468726a052d1dae 127.0.0.1:6380 master - 1318428930 1318428931 connected 1365-2729
    d289c575dcbc4bdd2931585fd4339089e461a27d 127.0.0.1:6381 master - 1318428931 1318428931 connected 2730-4095

在上面列出的三行信息中，
从左到右的各个域分别是：
节点 ID ，
IP 地址和端口号，
标志（flag），
最后发送 ``PING`` 的时间，
最后接收 ``PONG`` 的时间，
连接状态，
节点负责处理的槽。


节点握手（已实现）
-----------------------------------------------------------

节点总是应答（accept）来自集群连接端口的连接请求，
并对接收到的 ``PING`` 数据包进行回复，
即使这个 ``PING`` 数据包来自不可信的节点。

然而，
除了 ``PING`` 之外，
节点会拒绝其他所有并非来自集群节点的数据包。

要让一个节点承认另一个节点同属于一个集群，
只有以下两种方法：

- 一个节点可以通过向另一个节点发送 ``MEET`` 信息，
  来强制让接收信息的节点承认发送信息的节点为集群中的一份子。
  一个节点仅在管理员显式地向它发送 ``CLUSTER MEET ip port`` 命令时，
  才会向另一个节点发送 ``MEET`` 信息。

- 另外，
  如果一个可信节点向另一个节点传播第三者节点的信息，
  那么接收信息的那个节点也会将第三者节点识别为集群中的一份子。
  也即是说，
  如果 A 认识 B ，
  B 认识 C ，
  并且 B 向 A 传播关于 C 的信息，
  那么 A 也会将 C 识别为集群中的一份子，
  并尝试连接 C 。

这意味着如果我们将一个/一些新节点添加到一个集群中，
那么这个/这些新节点最终会和集群中已有的其他所有节点连接起来。

这说明只要管理员使用 ``CLUSTER MEET`` 命令显式地指定了可信关系，
集群就可以自动发现其他节点。

这种节点识别机制通过防止不同的 Redis 集群因为 IP 地址变更或者其他网络事件的发生而产生意料之外的联合（mix），
从而使得集群更具健壮性。

当节点的网络连接断开时，
它会主动连接其他已知的节点。


MOVED 转向
-----------------------------------------------------------

一个 Redis 客户端可以向集群中的任意节点（包括从节点）发送命令请求。
节点会对命令请求进行分析，
如果该命令是集群可以执行的命令，
那么节点会查找这个命令所要处理的键所在的槽。

如果要查找的哈希槽正好就由接收到命令的节点负责处理，
那么节点就直接执行这个命令。

另一方面，
如果所查找的槽不是由该节点处理的话，
节点将查看自身内部所保存的哈希槽到节点 ID 的映射记录，
并向客户端回复一个 ``MOVED`` 错误。

以下是一个 ``MOVED`` 错误的例子：

::

    GET x

    -MOVED 3999 127.0.0.1:6381

错误信息包含键 ``x`` 所属的哈希槽 ``3999`` ，
以及负责处理这个槽的节点的 IP 和端口号 ``127.0.0.1:6381`` 。
客户端需要根据这个 IP 和端口号，
向所属的节点重新发送一次 :ref:`GET` 命令请求。

注意，
即使客户端在重新发送 :ref:`GET` 命令之前，
等待了非常久的时间，
以至于集群又再次更改了配置，
使得节点 ``127.0.0.1:6381`` 已经不再处理槽 ``3999`` ，
那么当客户端向节点 ``127.0.0.1:6381`` 发送 :ref:`GET` 命令的时候，
节点将再次向客户端返回 ``MOVED`` 错误，
指示现在负责处理槽 ``3999`` 的节点。

虽然我们用 ID 来标识集群中的节点，
但是为了让客户端的转向操作尽可能地简单，
节点在 ``MOVED`` 错误中直接返回目标节点的 IP 和端口号，
而不是目标节点的 ID 。

虽然不是必须的，
但一个客户端应该记录（memorize）下“槽 ``3999`` 由节点 ``127.0.0.1:6381`` 负责处理“这一信息，
这样当再次有命令需要对槽 ``3999`` 执行时，
客户端就可以加快寻找正确节点的速度。

注意，
当集群处于稳定状态时，
所有客户端最终都会保存有一个哈希槽至节点的映射记录（map of hash slots to nodes），
使得集群非常高效：
客户端可以直接向正确的节点发送命令请求，
无须转向、代理或者其他任何可能发生单点故障（single point failure）的实体（entiy）。

除了 ``MOVED`` 转向错误之外，
一个客户端还应该可以处理稍后介绍的 ``ASK`` 转向错误。


集群在线重配置（live reconfiguration）
-----------------------------------------------------------

Redis 集群支持在集群运行的过程中添加或者移除节点。

实际上，
节点的添加操作和节点的删除操作可以抽象成同一个操作，
那就是，
将哈希槽从一个节点移动到另一个节点：

- 添加一个新节点到集群，
  等于将其他已存在节点的槽移动到一个空白的新节点里面。

- 从集群中移除一个节点，
  等于将被移除节点的所有槽移动到集群的其他节点上面去。

因此，
实现 Redis 集群在线重配置的核心就是将槽从一个节点移动到另一个节点的能力。
因为一个哈希槽实际上就是一些键的集合，
所以 Redis 集群在重哈希（rehash）时真正要做的，
就是将一些键从一个节点移动到另一个节点。

要理解 Redis 集群如何将槽从一个节点移动到另一个节点，
我们需要对 ``CLUSTER`` 命令的各个子命令进行介绍，
这些命理负责管理集群节点的槽转换表（slots translation table）。

以下是 ``CLUSTER`` 命令可用的子命令：

- ``CLUSTER ADDSLOTS slot1 [slot2] ... [slotN]``

- ``CLUSTER DELSLOTS slot1 [slot2] ... [slotN]``

- ``CLUSTER SETSLOT slot NODE node``

- ``CLUSTER SETSLOT slot MIGRATING node``

- ``CLUSTER SETSLOT slot IMPORTING node``

最开头的两条命令 ``ADDSLOTS`` 和 ``DELSLOTS`` 分别用于向节点指派（assign）或者移除节点，
当槽被指派或者移除之后，
节点会将这一信息通过 Gossip 协议传播到整个集群。
``ADDSLOTS`` 命令通常在新创建集群时，
作为一种快速地将各个槽指派给各个节点的手段来使用。

``CLUSTER SETSLOT slot NODE node`` 子命令可以将指定的槽 ``slot`` 指派给节点 ``node`` 。

至于 ``CLUSTER SETSLOT slot MIGRATING node`` 命令和 ``CLUSTER SETSLOT slot IMPORTING node`` 命令，
前者用于将给定节点 ``node`` 中的槽 ``slot`` 迁移出节点，
而后者用于将给定槽 ``slot`` 导入到节点 ``node`` ：

- 当一个槽被设置为 ``MIGRATING`` 状态时，
  原来持有这个槽的节点仍然会继续接受关于这个槽的命令请求，
  但只有命令所处理的键仍然存在于节点时，
  节点才会处理这个命令请求。

  如果命令所使用的键不存在与该节点，
  那么节点将向客户端返回一个 ``-ASK`` 转向（redirection）错误，
  告知客户端，
  要将命令请求发送到槽的迁移目标节点。

- 当一个槽被设置为 ``IMPORTING`` 状态时，
  节点仅在接收到 ``ASKING`` 命令之后，
  才会接受关于这个槽的命令请求。

  如果客户端没有向节点发送 ``ASKING`` 命令，
  那么节点会使用 ``-MOVED`` 转向错误将命令请求转向至真正负责处理这个槽的节点。

上面关于 ``MIGRATING`` 和 ``IMPORTING`` 的说明有些难懂，
让我们用一个实际的实例来说明一下。

假设现在，
我们有 A 和 B 两个节点，
并且我们想将槽 ``8`` 从节点 A 移动到节点 B ，
于是我们：

- 向节点 B 发送命令 ``CLUSTER SETSLOT 8 IMPORTING A``

- 向节点 A 发送命令 ``CLUSTER SETSLOT 8 MIGRATING B``

每当客户端向其他节点发送关于哈希槽 ``8`` 的命令请求时，
这些节点都会向客户端返回指向节点 A 的转向信息：

- 如果命令要处理的键已经存在于槽 ``8`` 里面，
  那么这个命令将由节点 A 处理。

- 如果命令要处理的键未存在于槽 ``8`` 里面（比如说，要向槽添加一个新的键），
  那么这个命令由节点 B 处理。

这种机制将使得节点 A 不再创建关于槽 ``8`` 的任何新键。

与此同时，
一个特殊的客户端 ``redis-trib`` 以及 Redis 集群配置程序（configuration utility）会将节点 A 中槽 ``8`` 里面的键移动到节点 B 。

键的移动操作由以下两个命令执行：

::

    CLUSTER GETKEYSINSLOT slot count

上面的命令会让节点返回 ``count`` 个 ``slot`` 槽中的键，
对于命令所返回的每个键，
``redis-trib`` 都会向节点 A 发送一条 :ref:`MIGRATE` 命令，
该命令会将所指定的键原子地（atomic）从节点 A 移动到节点 B 
（在移动键期间，两个节点都会处于阻塞状态，以免出现竞争条件）。

以下为 :ref:`MIGRATE` 命令的运作原理：

::

    MIGRATE target_host target_port key target_database id timeout

执行 :ref:`MIGRATE` 命令的节点会连接到 ``target`` 节点，
并将序列化后的 ``key`` 数据发送给 ``target`` ，
一旦 ``target`` 返回 ``OK`` ，
节点就将自己的 ``key`` 从数据库中删除。

从一个外部客户端的视角来看，
在某个时间点上，
键 ``key`` 要么存在于节点 A ，
要么存在于节点 B ，
但不会同时存在于节点 A 和节点 B 。

因为 Redis 集群只使用 ``0`` 号数据库，
所以当 :ref:`MIGRATE` 命令被用于执行集群操作时，
``target_database`` 的值总是 ``0`` 。

``target_database`` 参数的存在是为了让 :ref:`MIGRATE` 命令成为一个通用命令，
从而可以作用于集群以外的其他功能。

我们对 :ref:`MIGRATE` 命令做了优化，
使得它即使在传输包含多个元素的列表键这样的复杂数据时，
也可以保持高效。

不过，
尽管 :ref:`MIGRATE` 非常高效，
对一个键非常多、并且键的数据量非常大的集群来说，
集群重配置还是会占用大量的时间，
可能会导致集群没办法适应那些对于响应时间有严格要求的应用程序。


ASK 转向
-----------------------------------------------------------

在之前介绍 ``MOVED`` 转向的时候，
我们说除了 ``MOVED`` 转向之外，
还有另一种 ``ASK`` 转向。

当节点需要让一个客户端长期地（permanently）将针对某个槽的命令请求发送至另一个节点时，
节点向客户端返回 ``MOVED`` 转向。

另一方面，
当节点需要让客户端仅仅在下一个命令请求中转向至另一个节点时，
节点向客户端返回 ``ASK`` 转向。

比如说，
在我们上一节列举的槽 ``8`` 的例子中，
因为槽 ``8`` 所包含的各个键分散在节点 A 和节点 B 中，
所以当客户端在节点 A 中没找到某个键时，
它应该转向到节点 B 中去寻找，
但是这种转向应该仅仅影响一次命令查询，
而不是让客户端每次都直接去查找节点 B ：
在节点 A 所持有的属于槽 ``8`` 的键没有全部被迁移到节点 B 之前，
客户端应该先访问节点 A ，
然后再访问节点 B 。

因为这种转向只针对 ``16384`` 个槽中的其中一个槽，
所以转向对集群造成的性能损耗属于可接受的范围。

因为上述原因，
如果我们要在查找节点 A 之后，
继续查找节点 B ，
那么客户端在向节点 B 发送命令请求之前，
应该先发送一个 ``ASKING`` 命令，
否则这个针对带有 ``IMPORTING`` 状态的槽的命令请求将被节点 B 拒绝执行。

接收到客户端 ``ASKING`` 命令的节点将为客户端设置一个一次性的标志（flag），
使得客户端可以执行一次针对 ``IMPORTING`` 状态的槽的命令请求。

从客户端的角度来看，
``ASK`` 转向的完整语义（semantics）如下：

- 如果客户端接收到 ``ASK`` 转向，
  那么将命令请求的发送对象调整为转向所指定的节点。

- 先发送一个 ``ASKING`` 命令，然后再发送真正的命令请求。

- 不必更新客户端所记录的槽 ``8`` 至节点的映射：
  槽 ``8`` 应该仍然映射到节点 A ，
  而不是节点 B 。

一旦节点 A 针对槽 ``8`` 的迁移工作完成，
节点 A 在再次收到针对槽 ``8`` 的命令请求时，
就会向客户端返回 ``MOVED`` 转向，
将关于槽  ``8`` 的命令请求长期地转向到节点 B 。

注意，
即使客户端出现 Bug ，
过早地将槽 ``8`` 映射到了节点 B 上面，
但只要这个客户端不发送 ``ASKING`` 命令，
客户端发送命令请求的时候就会遇上 ``MOVED`` 错误，
并将它转向回节点 A 。


.. 
    客户端实现提示
    -----------------------------------------------------------

    TODO 流水线（Pipelining）: 使用 :ref:`MULTI` 和 :ref:`EXEC` 实现流水线。

    TODO 与节点建立持久（Persistent）连接。

    TODO 哈希槽猜测（guess）算法。


容错
-----------------------------------------------------------


节点失效检测
^^^^^^^^^^^^^^^^^^^^^^^^^

以下是节点失效检查的实现方法：

- 当一个节点向另一个节点发送 :ref:`PING` 命令，
  但是目标节点未能在给定的时限内返回 :ref:`PING` 命令的回复时，
  那么发送命令的节点会将目标节点标记为 ``PFAIL`` （possible failure，可能已失效）。

  等待 :ref:`PING` 命令回复的时限称为“节点超时时限（node timeout）”，
  是一个节点选项（node-wise setting）。

- 每次当节点对其他节点发送 :ref:`PING` 命令的时候，
  它都会随机地广播三个它所知道的节点的信息，
  这些信息里面的其中一项就是说明节点是否已经被标记为 ``PFAIL`` 或者 ``FAIL`` 。

- 当节点接收到其他节点发来的信息时，
  它会记下那些被其他节点标记为失效的节点。
  这称为失效报告（failure report）。

- 如果节点已经将某个节点标记为 ``PFAIL`` ，
  并且根据节点所收到的失效报告显式，
  集群中的大部分其他主节点也认为那个节点进入了失效状态，
  那么节点会将那个失效节点的状态标记为 ``FAIL`` 。

- 一旦某个节点被标记为 ``FAIL`` ，
  关于这个节点已失效的信息就会被广播到整个集群，
  所有接收到这条信息的节点都会将失效节点标记为 ``FAIL`` 。

简单来说，
一个节点要将另一个节点标记为失效，
必须先询问其他节点的意见，
并且得到大部分主节点的同意才行。

因为过期的失效报告会被移除，
所以主节点要将某个节点标记为 ``FAIL`` 的话，
必须以最近接收到的失效报告作为根据。

在以下两种情况中，
节点的 ``FAIL`` 状态会被移除：

- 如果被标记为 ``FAIL`` 的是从节点，
  那么当这个节点重新上线时，
  ``FAIL`` 标记就会被移除。

  保持（retaning）从节点的 ``FAIL`` 状态是没有意义的，
  因为它不处理任何槽，
  一个从节点是否处于 ``FAIL`` 状态，
  决定了这个从节点在有需要时能否被提升为主节点。

- 如果一个主节点被打上 ``FAIL`` 标记之后，
  经过了节点超时时限的四倍时间，
  再加上十秒钟之后，
  针对这个主节点的槽的故障转移操作仍未完成，
  并且这个主节点已经重新上线的话，
  那么移除对这个节点的 ``FAIL`` 标记。

在第二种情况中，
如果故障转移未能顺利完成，
并且主节点重新上线，
那么集群就继续使用原来的主节点，
从而免去管理员介入的必要。


集群状态检测（已部分实现）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

每当集群发生配置变化时（可能是哈希槽更新，也可能是某个节点进入失效状态），
集群中的每个节点都会对它所知道的节点进行扫描（scan）。

一旦配置处理完毕，
集群会进入以下两种状态的其中一种：

- ``FAIL`` ：
  集群不能正常工作。
  当集群中有某个节点进入失效状态时，
  集群不能处理任何命令请求，
  对于每个命令请求，
  集群节点都返回错误回复。

- ``OK`` ：
  集群可以正常工作，
  负责处理全部 ``16384`` 个槽的节点中，
  没有一个节点被标记为 ``FAIL`` 状态。

这说明即使集群中只有一部分哈希槽不能正常使用，
整个集群也会停止处理任何命令。

不过节点从出现问题到被标记为 ``FAIL`` 状态的这段时间里，
集群仍然会正常运作，
所以集群在某些时候，
仍然有可能只能处理针对 ``16384`` 个槽的其中一个子集的命令请求。

以下是集群进入 ``FAIL`` 状态的两种情况：

1) 至少有一个哈希槽不可用，因为负责处理这个槽的节点进入了 ``FAIL`` 状态。

2) 集群中的大部分主节点都进入下线状态。当大部分主节点都进入 ``PFAIL`` 状态时，集群也会进入 ``FAIL`` 状态。

第二个检查是必须的，
因为要将一个节点从 ``PFAIL`` 状态改变为 ``FAIL`` 状态，
必须要有大部分主节点进行投票表决，
但是，
当集群中的大部分主节点都进入失效状态时，
单凭一个两个节点是没有办法将一个节点标记为 ``FAIL`` 状态的。

因此，
有了第二个检查条件，
只要集群中的大部分主节点进入了下线状态，
那么集群就可以在不请求这些主节点的意见下，
将某个节点判断为 ``FAIL`` 状态，
从而让整个集群停止处理命令请求。


从节点选举
^^^^^^^^^^^^^^^^^^^

一旦某个主节点进入 ``FAIL`` 状态，
如果这个主节点有一个或多个从节点存在，
那么其中一个从节点会被升级为新的主节点，
而其他从节点则会开始对这个新的主节点进行复制。

新的主节点由已下线主节点属下的所有从节点中自行选举产生，
以下是选举的条件：

- 这个节点是已下线主节点的从节点。

- 已下线主节点负责处理的槽数量非空。

- 从节点的数据被认为是可靠的，
  也即是，
  主从节点之间的复制连接（replication link）的断线时长不能超过节点超时时限（node timeout）乘以 ``REDIS_CLUSTER_SLAVE_VALIDITY_MULT`` 常量得出的积。

如果一个从节点满足了以上的所有条件，
那么这个从节点将向集群中的其他主节点发送授权请求，
询问它们，
是否允许自己（从节点）升级为新的主节点。

如果发送授权请求的从节点满足以下属性，
那么主节点将向从节点返回 ``FAILOVER_AUTH_GRANTED`` 授权，
同意从节点的升级要求：

- 发送授权请求的是一个从节点，
  并且它所属的主节点处于 ``FAIL`` 状态。

- 在已下线主节点的所有从节点中，
  这个从节点的节点 ID 在排序中是最小的。

- 这个从节点处于正常的运行状态：
  它没有被标记为 ``FAIL`` 状态，
  也没有被标记为 ``PFAIL`` 状态。

一旦某个从节点在给定的时限内得到大部分主节点的授权，
它就会开始执行以下故障转移操作：

- 通过 ``PONG`` 数据包（packet）告知其他节点，
  这个节点现在是主节点了。

- 通过 ``PONG`` 数据包告知其他节点，
  这个节点是一个已升级的从节点（promoted slave）。

- 接管（claiming）所有由已下线主节点负责处理的哈希槽。

- 显式地向所有节点广播一个 ``PONG`` 数据包，
  加速其他节点识别这个节点的进度，
  而不是等待定时的 ``PING`` / ``PONG`` 数据包。

所有其他节点都会根据新的主节点对配置进行相应的更新，特别地：

- 所有被新的主节点接管的槽会被更新。

- 已下线主节点的所有从节点会察觉到 ``PROMOTED`` 标志，
  并开始对新的主节点进行复制。

- 如果已下线的主节点重新回到上线状态，
  那么它会察觉到 ``PROMOTED`` 标志，
  并将自身调整为现任主节点的从节点。

在集群的生命周期中，
如果一个带有 ``PROMOTED`` 标识的主节点因为某些原因转变成了从节点，
那么该节点将丢失它所带有的 ``PROMOTED`` 标识。


发布/订阅（已实现，但仍然需要改善）
--------------------------------------------------------

在一个 Redis 集群中，
客户端可以订阅任意一个节点，
也可以向任意一个节点发送信息，
节点会对客户端所发送的信息进行转发。

在目前的实现中，
节点会将接收到的信息广播至集群中的其他所有节点，
在将来的实现中，
可能会使用 bloom filter 或者其他算法来优化这一操作。


附录 A： CRC16 算法的 ANSI 实现参考
----------------------------------------------------------

::

    /*
     * Copyright 2001-2010 Georges Menie (www.menie.org)
     * Copyright 2010 Salvatore Sanfilippo (adapted to Redis coding style)
     * All rights reserved.
     * Redistribution and use in source and binary forms, with or without
     * modification, are permitted provided that the following conditions are met:
     *
     *     * Redistributions of source code must retain the above copyright
     *       notice, this list of conditions and the following disclaimer.
     *     * Redistributions in binary form must reproduce the above copyright
     *       notice, this list of conditions and the following disclaimer in the
     *       documentation and/or other materials provided with the distribution.
     *     * Neither the name of the University of California, Berkeley nor the
     *       names of its contributors may be used to endorse or promote products
     *       derived from this software without specific prior written permission.
     *
     * THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND ANY
     * EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
     * WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
     * DISCLAIMED. IN NO EVENT SHALL THE REGENTS AND CONTRIBUTORS BE LIABLE FOR ANY
     * DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
     * (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
     * LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
     * ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
     * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
     * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
     */

    /* CRC16 implementation acording to CCITT standards.
     *
     * Note by @antirez: this is actually the XMODEM CRC 16 algorithm, using the
     * following parameters:
     *
     * Name                       : "XMODEM", also known as "ZMODEM", "CRC-16/ACORN"
     * Width                      : 16 bit
     * Poly                       : 1021 (That is actually x^16 + x^12 + x^5 + 1)
     * Initialization             : 0000
     * Reflect Input byte         : False
     * Reflect Output CRC         : False
     * Xor constant to output CRC : 0000
     * Output for "123456789"     : 31C3
     */

    static const uint16_t crc16tab[256]= {
        0x0000,0x1021,0x2042,0x3063,0x4084,0x50a5,0x60c6,0x70e7,
        0x8108,0x9129,0xa14a,0xb16b,0xc18c,0xd1ad,0xe1ce,0xf1ef,
        0x1231,0x0210,0x3273,0x2252,0x52b5,0x4294,0x72f7,0x62d6,
        0x9339,0x8318,0xb37b,0xa35a,0xd3bd,0xc39c,0xf3ff,0xe3de,
        0x2462,0x3443,0x0420,0x1401,0x64e6,0x74c7,0x44a4,0x5485,
        0xa56a,0xb54b,0x8528,0x9509,0xe5ee,0xf5cf,0xc5ac,0xd58d,
        0x3653,0x2672,0x1611,0x0630,0x76d7,0x66f6,0x5695,0x46b4,
        0xb75b,0xa77a,0x9719,0x8738,0xf7df,0xe7fe,0xd79d,0xc7bc,
        0x48c4,0x58e5,0x6886,0x78a7,0x0840,0x1861,0x2802,0x3823,
        0xc9cc,0xd9ed,0xe98e,0xf9af,0x8948,0x9969,0xa90a,0xb92b,
        0x5af5,0x4ad4,0x7ab7,0x6a96,0x1a71,0x0a50,0x3a33,0x2a12,
        0xdbfd,0xcbdc,0xfbbf,0xeb9e,0x9b79,0x8b58,0xbb3b,0xab1a,
        0x6ca6,0x7c87,0x4ce4,0x5cc5,0x2c22,0x3c03,0x0c60,0x1c41,
        0xedae,0xfd8f,0xcdec,0xddcd,0xad2a,0xbd0b,0x8d68,0x9d49,
        0x7e97,0x6eb6,0x5ed5,0x4ef4,0x3e13,0x2e32,0x1e51,0x0e70,
        0xff9f,0xefbe,0xdfdd,0xcffc,0xbf1b,0xaf3a,0x9f59,0x8f78,
        0x9188,0x81a9,0xb1ca,0xa1eb,0xd10c,0xc12d,0xf14e,0xe16f,
        0x1080,0x00a1,0x30c2,0x20e3,0x5004,0x4025,0x7046,0x6067,
        0x83b9,0x9398,0xa3fb,0xb3da,0xc33d,0xd31c,0xe37f,0xf35e,
        0x02b1,0x1290,0x22f3,0x32d2,0x4235,0x5214,0x6277,0x7256,
        0xb5ea,0xa5cb,0x95a8,0x8589,0xf56e,0xe54f,0xd52c,0xc50d,
        0x34e2,0x24c3,0x14a0,0x0481,0x7466,0x6447,0x5424,0x4405,
        0xa7db,0xb7fa,0x8799,0x97b8,0xe75f,0xf77e,0xc71d,0xd73c,
        0x26d3,0x36f2,0x0691,0x16b0,0x6657,0x7676,0x4615,0x5634,
        0xd94c,0xc96d,0xf90e,0xe92f,0x99c8,0x89e9,0xb98a,0xa9ab,
        0x5844,0x4865,0x7806,0x6827,0x18c0,0x08e1,0x3882,0x28a3,
        0xcb7d,0xdb5c,0xeb3f,0xfb1e,0x8bf9,0x9bd8,0xabbb,0xbb9a,
        0x4a75,0x5a54,0x6a37,0x7a16,0x0af1,0x1ad0,0x2ab3,0x3a92,
        0xfd2e,0xed0f,0xdd6c,0xcd4d,0xbdaa,0xad8b,0x9de8,0x8dc9,
        0x7c26,0x6c07,0x5c64,0x4c45,0x3ca2,0x2c83,0x1ce0,0x0cc1,
        0xef1f,0xff3e,0xcf5d,0xdf7c,0xaf9b,0xbfba,0x8fd9,0x9ff8,
        0x6e17,0x7e36,0x4e55,0x5e74,0x2e93,0x3eb2,0x0ed1,0x1ef0
    };

    uint16_t crc16(const char *buf, int len) {
        int counter;
        uint16_t crc = 0;
        for (counter = 0; counter < len; counter++)
                crc = (crc<<8) ^ crc16tab[((crc>>8) ^ *buf++)&0x00FF];
        return crc;
    }

