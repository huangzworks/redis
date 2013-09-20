Sentinel
============

.. warning::

    Sentinel 系统目前正处于开发状态，
    本文档中的描述可能会随着 Sentinel 实现的演进而变化，
    敬请知悉。

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance），
该系统执行以下三个任务：

- 监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否正常运行。

- 提醒（Notification）：当被监控的某个 Redis 服务器出现问题时，
  Sentinel 可以向系统管理员发送通知，
  也可以通过 API 向其他程序发送通知。

- 自动故障转移（Automatic failover）：
  当一个主服务器不能正常工作时，
  Sentinel 可以将一个从服务器升级为主服务器，
  并对其他从服务器进行配置，
  让它们使用新的主服务器。
  当应用程序连接到 Redis 服务器时，
  它们会被告知服务器的地址已经变更。

.. - Automatic failover. If a master is not working as expected, Sentinel can start a failover process where a slave is promoted to master, the other additional slaves are reconfigured to use the new master, and the applications using the Redis server informed about the new address to use when connecting.

Redis Sentinel 是一个分布式系统，
你可以在架构中运行多个 Sentinel 进程，
这些 Sentinel 进程通过相互通讯来判断一个主服务器是否断线，
以及是否应该执行故障转移。

虽然 Redis Sentinel 释出为一个单独的可执行文件 ``redis-sentinel`` ，
但实际上它只是一个运行在特殊模式下的 Redis 服务器，
你可以在启动一个普通 Redis 服务器时通过给定 ``--sentinel`` 选项来启动 Redis Sentinel 。

Redis Sentinel 适用于 Redis 2.4.16 或以上版本，以及 Redis 2.6.0-rc6 以上版本。


获取 Sentinel
---------------------

目前 Sentinel 系统是 Redis 的 ``unstable`` 分支的一部分，
你必须到 `Redis 项目的 Github 页面  <https://github.com/antirez/redis>`_ 克隆一份 ``unstable`` 分值，
然后通过编译来获得 Sentinel 系统。

Sentinel 程序可以在编译后的 ``src`` 文档中发现，
它是一个命名为 ``redis-sentinel`` 的程序。

另外，
你也可以通过下一节介绍的方法，
让 ``redis-server`` 程序运行在 Sentinel 模式之下。


启动 Sentinel
----------------------

对于 ``redis-sentinel`` 程序，
你可以用以下命令来启动 Sentinel 系统：

::

    redis-sentinel /path/to/sentinel.conf

对于 ``redis-server`` 程序，
你可以用以下命令来启动服务器的 Sentinel 模式：

::

    redis-server /path/to/sentinel.conf --sentinel

它们的效果是一样的。


配置 Sentinel
---------------------

Redis 源码中包含了一个 ``sentinel.conf`` 文件，
该文件是一个带有详细注释的 Sentinel 配置文件示例。

以下是另一个 Sentinel 配置文件示例，
它包含了运行 Sentinel 系统所需的最少配置：

::

    sentinel monitor mymaster 127.0.0.1 6379 2
    sentinel down-after-milliseconds mymaster 60000
    sentinel failover-timeout mymaster 900000
    sentinel can-failover mymaster yes
    sentinel parallel-syncs mymaster 1

    sentinel monitor resque 192.168.1.3 6380 4
    sentinel down-after-milliseconds resque 10000
    sentinel failover-timeout resque 900000
    sentinel can-failover resque yes
    sentinel parallel-syncs resque 5

第一行配置文件告诉 Sentinel 系统，
让它监视一个名为 ``mymaster`` 的主服务器，
该服务器的地址为 ``127.0.0.1`` ，
端口号为 ``6379`` ，
之后的 ``2`` 表示将这个主服务器识别为失效所需的 Sentinel 数量：
至少要有两个 Sentinel 都认为这个主服务器失效时，
自动故障转移操作才会启动。

其他选项的基本格式如下：

::

    sentinel <选项的名字> <主服务器的名字> <选项的值>

各个选项的功能如下：

- ``down-after-milliseconds`` 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。

  如果服务器在给定的毫秒数之内，
  没有返回 Sentinel 发送的 :ref:`PING` 命令的回复，
  或者返回一个错误，
  那么 Sentinel 将这个服务器标记为主观下线（subjectively down，简称 ``SDOWN`` ）。

  不过只有一个 Sentinel 将服务器标记为主观下线并不一定会引起服务器的自动故障转移：
  只有在足够数量的 Sentinel 都将一个服务器标记为主观下线之后，
  服务器才会被标记为客观下线（objectively down， 简称 ``ODOWN`` ），
  这时自动故障迁移才会执行。

  将服务器标记为客观下线所需的 Sentinel 数量由对主服务器的配置决定。

- ``can-failover`` 选项指定 Sentinel 是否需要在服务器被标记为 ``ODOWN`` 时，
  对服务器执行故障转移。

  通过这个选项，
  你可以让所有 Sentinel 都有执行故障转移的权力，
  也可以让一部分 Sentinel 单纯用于判断服务器是否下线，
  而另一部分 Sentinel 则负责执行故障转移。

- ``parallel-syncs`` 选项指定了在执行故障转移时，
  最多可以有多少个从服务器同时对新的主服务器进行同步，
  这个数字越小，
  完成故障转移所需的时间就越长。

  如果从服务器被设置为允许使用过期数据集（参见对 ``redis.conf`` 文件中对 ``slave-serve-stale-data`` 选项的说明），
  那么你可能不希望所有从服务器都在同一时间向新的主服务器发送同步请求，
  因为尽管复制过程的绝大部分步骤都不会阻塞从服务器，
  但从服务器在载入主服务器发来的 RDB 文件时，
  仍然会造成从服务器在一段时间内不能处理命令请求：
  如果全部从服务器一起对新的主服务器进行同步，
  那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。

  你可以通过将这个值设为 ``1`` 来保证每次只有一个从服务器处于不能处理命令请求的状态。

本文档剩余的内容将对 Sentinel 系统的其他选项进行介绍，
``sentinel.conf`` 文件中也对相关的选项进行了完整的注释。


主观下线和客观下线
--------------------------------

前面简单介绍过，
Redis 的 Sentinel 中关于下线（down）有两个不同的概念：

- 主观下线（Subjectively Down， 简称 SDOWN）是单个 Sentinel 实例做出的对某个服务器的下线判断。

- 客观下线（Objectively Down， 简称 ODOWN）是多个 Sentinel 实例在做出主观下线判断之后，
  得出来某个服务器已经下线的结论。

  一个 Sentinel 可以通过 ``SENTINEL is-master-down-by-addr`` 命令来询问另一个 Sentinel ，
  查看被询问的 Sentinel 是否认为给定服务器已经下线。

从 Sentinel 的角度来看，
当一个服务器没有在 ``master-down-after-milliseconds`` 选项所指定的时间内对 Sentinel 所发送的 :ref:`PING` 命令做出正确的（valid）回复时，
该服务器就会被标记为主观下线。

以下列举的是 :ref:`PING` 命令的正确回复：

- 服务器返回 ``+PONG`` 。

- 服务器返回 ``-LOADING`` 错误。

- 服务器返回 ``-MASTERDOWN`` 错误。

如果服务器返回其他回复，
或者服务器不回复 :ref:`PING` 命令，
都会被认为是不正确的回复。

一个服务器必须在给定的时间间隔内，
一直返回不正确回复才会被 Sentinel 标记为主观下线。

举个例子，
如果 ``master-down-after-milliseconds`` 选项的值为 ``30000`` 毫秒（\ ``30`` 秒），
那么只要服务器能在每 ``29`` 秒之内返回至少一次正确回复，
这个服务器就仍然会被认为是处于正常状态的。

客观下线条件\ **只适用于主服务器**\ ：
对于任何其他类型的 Redis 实例，
Sentinel 不需要协商如何处理它们，
所以从服务器或者其他 Sentinel 永远不会达到客观下线条件。

Redis Sentinel 的行为可以用一集所有 Sentinel 都遵循的规则来描述，
多个 Sentinel 所组成的分布式 Sentinel 系统的行为也是从单个 Sentinel 的规则中衍生而出的。

以下是 Sentinel 的第一部分规则，
在文档之后的内容中，
我们还会介绍其他规则：

**Sentinel 规则 #1**\ ：每个 Sentinel 每秒都向它所知的主服务器、从服务器以及其他 Sentinel 实例发送一个 :ref:`PING` 命令。

**Sentinel 规则 #2**\ ：如果一个实例（instance）距离最后一次正确回复 :ref:`PING` 命令的时间超过 ``down-after-milliseconds`` 选项所指定的值，
那么这个实例会被 Sentinel 标记为主观下线。
一个正确的回复可以是： ``+PONG`` 、 ``-LOADING`` 或者 ``-MASTERDOWN`` 。

**Sentinel 规则 #3**\ ：
每个 Sentinel 都可以接受 ``SENTINEL is-master-down-by-addr <ip> <port>`` 命令，
如果给定的地址是一个主服务器，
并且该服务器已经被 Sentinel 标记为主观下线，
那么 Sentinel 向发送这个命令的实例返回真。

**Sentinel 规则 #4**\ ：
如果一个主服务器被标记为主观下线，
那么正在监视这个主服务器的所有 Sentinel 都必须每秒发送一次 ``SENTINEL is-master-down-by-addr`` 命令，
检查这一判断是否正确。

**Sentinel 规则 #5**\ ：
如果有足够数量的 Sentinel 都将同一个主服务器标记为主观下线，
并且这些 Sentinel 所发送的最后一条 ``SENTINEL is-master-down-by-addr`` 命令回复不超过 5 秒钟，
那么这个主服务器会被标记为客观下线。

.. note::

    译注：“Sentinel 所发送的最后一条 ``SENTINEL is-master-down-by-addr`` 命令回复不超过 5 秒钟”指的是，
    某个 Sentinel 在 5 秒钟之内对主服务器进行了一次检测，
    并且主服务器的回复仍然是不正确的。

**Sentinel 规则 #6**\ ：
一般情况下，
每个 Sentinel 每 10 秒向它所知道的每个主服务器和从服务器发送一个 :ref:`INFO` 命令。
当一个主服务器被 Sentinel 标记为主观下线时，
Sentinel 向这个主服务器的所有从服务器发送 :ref:`INFO` 命令的频率会从 10 秒一次改为每秒一次。

**Sentinel 规则 #7**\ ：
如果启动 Sentinel 时指定的主服务器实际上是从服务器，
那么 Sentinel 会根据从服务器返回的 :ref:`INFO` 命令的信息，
自动将监视的目标改为从服务器所属的主服务器。
所以对从服务器启动 Sentinel 也是安全的。


自动发现 Sentinel 和从服务器
--------------------------------

一个 Sentinel 可以连接其他多个 Sentinel ，
各个 Sentinel 之间可以互相检查对方的可用性，
并进行信息交换。

你无须对你所运行的每个 Sentinel 设置其他 Sentinel 的地址，
因为 Sentinel 可以使用发布与订阅功能，
通过发送信息的方法来自动发现其他正在监视相同主服务器的 Sentinel 。

这一功能是通过向频道 ``__sentinel__:hello`` 发送信息来实现的。

与此类似，
你也不必手动列出主服务器属下的所有从服务器，
因为 Sentinel 可以通过询问主服务器来获得所有从服务器的信息。

**Sentinel 规则 #8**\ ：
每个 Sentinel 每 5 秒就会向它所监视的主服务器的 ``__sentinel__:hello`` 频道发送一条信息，
信息的内容包括这个 Sentinel 的 IP 地址、端口号、运行 ID （runid），
以及是否可以进行故障转移（由 ``sentinel.conf`` 中的 ``can-failover`` 选项决定）。

**Sentinel 规则 #9**\ ：
每个 Sentinel 都会订阅它所监视的主服务器的 ``__sentinel__:hello`` 频道，
等待新的 Sentinel 出现。
当发现新 Sentinel 时，
原来的 Sentinel 会将这个新 Sentinel 加入到对主服务器的监视工作中。

**Sentinel 规则 #10**\ ：
在添加一个 Sentinel 之前，
已有的 Sentinel 会对这个 Sentinel 的运行 ID 、IP 地址和端口进行检查，
只有在该 Sentinel 尚未存在的情况下，
已有的 Sentinel 才会让这个新 Sentinel 开始对主服务器进行监视。


Sentinel API
---------------

在默认情况下，
Sentinel 使用 TCP 端口 ``26379`` 。

Sentinel 接受 Redis 协议的命令请求，
所以你可以使用 ``redis-cli`` ，
或者任何其他 Redis 客户端来与 Sentinel 进行通讯。

有两种方式可以和 Sentinel 进行通讯：

- 第一种方法是通过直接发送命令来查询被监视 Redis 服务器的当前状态，
  以及 Sentinel 所知道的关于其他 Sentinel 的信息，
  诸如此类。

- 另一种方法是使用发布与订阅功能，
  通过接收 Sentinel 发送的通知：
  当执行故障转移操作，
  或者某个被监视的服务器被判断为主观下线或者客观下线时，
  Sentinel 就会发送相应的信息。


Sentinel 命令
^^^^^^^^^^^^^^^^^^^^^

以下列出的是 Sentinel 接受的命令：

- :ref:`PING` ：返回 ``PONG`` 。

- ``SENTINEL masters`` ：列出所有被监视的主服务器，以及它们的状态。

- ``SENTINEL slaves <master name>`` ：列出给定主服务器的所有从服务器，以及它们的状态。

- ``SENTINEL is-master-down-by-addr <ip> <port>`` ：
  返回一个包含两个元素的 multi bulk reply 。 
  第一个元素的值可以是 ``0`` 或者 ``1`` ， ``0`` 表示给定的主服务器处于主观下线状态， ``1`` 则表示主服务器正常运行中。
  第二个元素则是给定主服务器的主观领袖（subjective leader），
  也即是，
  当需要对给定主服务器执行故障转移操作时，
  负责执行故障转移操作的 Sentinel 实例的运行 ID 。

- ``SENTINEL get-master-addr-by-name <master name>`` ：
  返回给定名字的主服务器的 IP 地址和端口号。
  如果这个主服务器正在执行故障转移操作，
  或者故障转移操作已经成功完成，
  那么这个命令返回新的主服务器的 IP 地址和端口号。

- ``SENTINEL reset <pattern>`` ：
  重置所有名字和给定模式 ``pattern`` 相匹配的主服务器。
  ``pattern`` 参数是一个 Glob 风格的模式。
  重置操作清楚主服务器目前的所有状态，
  包括正在执行中的故障转移，
  并移除目前已经发现和关联的，
  主服务器的所有从服务器和 Sentinel 。
  

发布与订阅信息
^^^^^^^^^^^^^^^^^^^^^

客户端可以将 Sentinel 看作是一个只提供了订阅功能的 Redis 服务器：
你不可以使用 :ref:`PUBLISH` 命令向这个服务器发送信息，
但你可以用 :ref:`SUBSCRIBE` 命令或者 :ref:`PSUBSCRIBE` 命令，
通过订阅给定的频道来获取相应的事件提醒。

一个频道能够接收和这个频道的名字相同的事件。
比如说，
名为 ``+sdown`` 的频道就可以接收所有实例进入主观下线（SDOWN）状态的事件。

通过执行 ``PSUBSCRIBE *`` 命令可以接收所有事件信息。

以下列出的是客户端可以通过订阅来获得的频道和信息的格式：
第一个英文单词是频道/事件的名字，
其余的是数据的格式。

注意，
当格式中包含 ``instance details`` 字样时，
表示频道所返回的信息中包含了以下用于识别目标实例的内容：

::

    <instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>

``@`` 字符之后的内容用于指定主服务器，
这些内容是可选的，
它们仅在 ``@`` 字符之前的内容指定的实例不是主服务器时使用。

- ``+reset-master <instance details>`` ：主服务器已被重置。

- ``+slave <instance details>`` ：一个新的从服务器已经被 Sentinel 识别并关联。

- ``+failover-state-reconf-slaves <instance details>`` ：故障转移状态切换到了 ``reconf-slaves`` 状态。

- ``+failover-detected <instance details>`` ：另一个 Sentinel 开始了一次故障转移操作，或者一个从服务器转换成了主服务器。

- ``+slave-reconf-sent <instance details>`` ：领头（leader）的 Sentinel 向实例发送了 :ref:`SLAVEOF` 命令，为实例设置新的主服务器。

- ``+slave-reconf-inprog <instance details>`` ：实例正在将自己设置为指定主服务器的从服务器，但相应的同步过程仍未完成。

- ``+slave-reconf-done <instance details>`` ：从服务器已经成功完成对新主服务器的同步。

- ``-dup-sentinel <instance details>`` ：对给定主服务器进行监视的一个或多个 Sentinel 已经因为重复出现而被移除 —— 当 Sentinel 实例重启的时候，就会出现这种情况。

- ``+sentinel <instance details>`` ：一个监视给定主服务器的新 Sentinel 已经被识别并添加。

- ``+sdown <instance details>`` ：给定的实例现在处于主观下线状态。

- ``-sdown <instance details>`` ：给定的实例已经不再处于主观下线状态。

- ``+odown <instance details>`` ：给定的实例现在处于客观下线状态。

- ``-odown <instance details>`` ：给定的实例已经不再处于客观下线状态。

- ``+failover-takedown <instance details>`` ：如果设定的故障转移时间已经过去 25% ，并且故障转移工作没有任何进展的话，那么给定的 Sentinel 就会成为新的领头 Sentinel ，并使用剩余的从服务器来创建新的主服务器。

- ``+failover-triggered <instance details>`` ：给定的 Sentinel 作为领头 Sentinel 开始了一次新的故障转移操作。

- ``+failover-state-wait-start <instance details>`` ：故障转移操作现在处于 ``wait-start`` 状态 —— Sentinel 在启动一次故障转移前，会先等待一段固定长度的时间（以秒为单位），然后再等待一段随机长度的时间（以秒为单位）。

- ``+failover-state-select-slave <instance details>`` ：故障转移操作现在处于 ``select-slave`` 状态 —— Sentinel 正在寻找可以升级为主服务器的从服务器。

- ``no-good-slave <instance details>`` ：Sentinel 操作未能找到适合进行升级的从服务器。Sentinel 会在一段时间之后再次尝试寻找合适的从服务器来进行升级，又或者直接放弃执行故障转移操作。

- ``selected-slave <instance details>`` ：Sentinel 顺利找到适合进行升级的从服务器。

- ``failover-state-send-slaveof-noone <instance details>`` ：Sentinel 正在将指定的从服务器升级为主服务器，等待升级功能完成。

- ``failover-end-for-timeout <instance details>`` ：故障转移因为超时而中止。如果当前 Sentinel 为领头 Sentinel ，那么它将向所有正在等待重配置（reconfigure）的从服务器发送 :ref:`SLAVEOF` 命令。其中， :ref:`SLAVEOF` 命令所设置的主服务器是 Sentinel 目前能找到的最佳主服务器候选。

- ``failover-end <instance details>`` ：故障转移操作顺利完成。所有等待重配置的从服务器将开始与新的主服务器进行同步。

- ``switch-master <master name> <oldip> <oldport> <newip> <newport>`` ：Sentinel 开始监控新的主服务器，至于主服务器的名字，则沿用旧的主服务器的名字。旧的主服务器会从 Sentinel 的监控列表中被移除。

- ``failover-abort-x-sdown <instance details>`` ：故障转移操作已经被中止，因为原来打算升级为主服务器的从服务器已经被标记为主观下线状态。
 
- ``-slave-reconf-undo <instance details>`` ：故障转移操作已经被中止，Sentinel 向给定的实例发送 :ref:`SLAVEOF` 命令，让它继续使用原来的主服务器。

- ``+tilt`` ：进入 tilt 模式。

- ``-tilt`` ：退出 tilt 模式。


故障转移
--------------------

故障转移过程包含以下三个步骤：

- 识别出已经进入客观下线状态的主服务器。

- 找出负责对已下线主服务器进行故障转移操作的 Sentinel 。
  负责故障转移的 Sentinel 称为领头 Sentinel ，
  其他的 Sentinel 都是观察者（Observer) Sentinel 。

- 领头 Sentinel 找出可以升级为主服务器的从服务器。

- 从服务器通过执行 ``SLAVEOF NO ONE`` 命令来将自己转换成主服务器。

- 观察者看见一个从服务器变成了主服务器，于是它们知道一次故障转移开始了。
  注意，
  这说明，
  当一个被监视的主服务器的某个从服务器升级为主服务器时，
  这意味着一次故障转移开始了。

- 旧主服务器的所有从服务器都会通过 :ref:`SLAVEOF` 命令， 
  开始与新主服务器进行同步。

- 当所有从服务器都完成了与新主服务器的同步之后，
  领头 Sentinel 停止故障转移操作，
  并将旧主服务器从监视主服务器的列表中删除，
  然后将新主服务器添加到监视列表中，
  监视列表中新主服务器的名字沿用旧服务器的名字。

- 当故障转移操作完成之后，观察者 Sentinel 也会将旧主服务器从监视列表中移除，
  然后开始监视新主服务器，
  正如领头 Sentinel 所做的那样。

挑选领头 Sentinel 的机制和将一个实例判断为客观下线所使用的机制一样，
它们都是由 ``SENTINEL is-master-down-by-addr`` 命令来实现的：
一个 Sentinel 在接到这个命令的时候，
会向发送命令的实例返回这个 Sentinel 所认为的领头 Sentinel ，
这个领头 Sentinel 被称为主观领头（Subjective Leader） Sentinel ，
以下是挑选主观领头 Sentinel 的方法：

- 系统首先将不能执行故障转移的 Sentinel 从筛选列表中移除
  （一个 Sentinel 会通过发送信息到频道来说明自己能否执行故障转移）。

- 系统从筛选列表中移除所有处于以下状态的 Sentinel ：
  Sentinel 被标记为主观下线，
  Sentinel 已断线，
  或者 Sentinel 最后回复 :ref:`PING` 命令的时间超过 ``SENTINEL_INFO_VALIDITY_TIME`` 选项所指定的毫秒数
  （目前定位为 ``5`` 秒）。

- 在所有筛选剩下的 Sentinel 实例里面，
  系统按字典序对实例的运行 ID 进行排序，
  并选出运行 ID 最小的那个 Sentinel 实例作为主观领头 Sentinel 。

一个 Sentinel 要成为客观领头 Sentinel ，
也即是，
负责执行故障转移操作的 Sentinel ，
它必须满足以下条件：

- 这个 Sentinel 认为（think）自己是主观领头 Sentinel 。

- 其他 Sentinel 也承认这个 Sentinel 为领头 Sentinel ：
  对于所有可以回复 ``SENTINEL is-master-down-by-addr`` 命令的 Sentinel 来说，
  这些 Sentinel 总数的 50% 以上同意这个 Sentinel 为领头 Sentinel ，
  并且同意 Sentinel 的数量大于或等于配置所设定的，
  执行故障转移所需的最少 Sentinel 数量。

一旦一个 Sentinel 满足了以上列举的条件，
那么它就是一个客观领头 Sentinel ，
这个 Sentinel 可以开始执行故障转移操作。

不过，
在开始执行故障转移操作之前，
程序总会延迟五秒加上一个随机秒数之后，
才真正开始执行故障转移操作。

这是一种额外的保护措施，
如果在延迟期间，
客观领头 Sentinel 看见有其他 Sentinel 实例将一个从服务器升级成了主服务器，
那么这个客观领头 Sentinel 就会知道已经有其他客观领头 Sentinel 开始了一次故障转移操作，
并将这个客观领头 Sentinel 转变为观察者 Sentinel 。

因为理论上不可能同时出现两个客观领头 Sentinel ，
所以这个保护措施只是一个冗余措施。

**Sentinel 规则 #11**\ ：
一个状况良好的从服务器应该能满足以下条件：

- 从服务器既不处于主观下线状态，也不处于客观下线状态。

- 从服务器目前的网络链接状况良好，没有断线。

- 从服务器最近一次返回 :ref:`PING` 命令的时间，距离现在时间不超过五秒钟。

- 从服务器最近一次返回 :ref:`INFO` 命令的时间，距离现在时间不超过五秒钟。

- 根据最近一次 :ref:`INFO` 命令返回的信息，
  主从服务器之间的断线时长，
  不超过 Sentinel 发现主服务器进入主观下线状态的时长，
  加上 ``down_after_milliseconds`` 参数的值乘以 ``10`` 。

  比如说，
  如果 Sentinel 被配置为将断线 ``10`` 秒钟的主服务器标记为主观下线，
  并且主服务器已经下线 ``50`` 秒，
  那么我们只将和主服务器断开连接少于 ``50+(10*10)`` 秒的从服务器看作是状态良好的从服务器。

- 从服务器没有被标记为 ``DEMOTE`` （参考后面的《主服务器恢复》一节）。

**Sentinel 规则 #12**\ ：
一个主观领头 Sentinel ，
就是所有监视同一主服务器中，
运行 ID 较低的 Sentinel ；
并且这个 Sentinel 最近一次返回 :ref:`PING` 命令的时间距离当前时间不超过 5 秒钟；
并且这个 Sentinel 通过发送信息告知了其他 Sentinel ，
它可以执行故障转移操作；
最后，
这个 Sentinel 不处于 ``DISCONNECTED`` 状态。

**Sentinel 规则 #12**\ ：
当一个主服务器下线时，
根据 Sentinel 规则 #4 所说的，
Sentinel 向所有其他 Sentinel 发送 ``SENTINEL is-master-down-by-addr`` 。

接收命令的 Sentinel 会向发送命令的 Sentinel 返回自己的主观领头 Sentinel 的运行 ID 。

当一个 Sentinel 被包括它自己在内的至少 ``N`` 个 Sentinel 承认为主观领头 Sentinel 时，
这个 Sentinel 就成为了给定主服务器的客观领头 Sentinel ，
其中：

- ``N`` 必须大于或等于配置所设定的，执行故障转移所需的最少 Sentinel 数量。

- 对于同样报告主服务器已经下线的所有 Sentinel 来说，
  认同这个 Sentinel 为主观领头 Sentinel 的 Sentinel 数量必须大于半数。
  也即是，
  对于投票数 ``num_votes`` 来说，
  ``N`` 必须大于等于 ``num_votes/2+1`` 。

**Sentinel 规则 #13**\ ：

当以下条件都同时满足的时候，
客观领头 Sentinel 开始执行故障转移操作，
也即是，
该 Sentinel 向下线主服务器的所有从服务器发送重配置（reconfigure）命令：
    
- 主服务器处于客观下线状态。

- 该 Sentinel 的 ``can-failover`` 选项为 ``yes`` 。

- 该 Sentinel 目前发现至少有一个状态良好的从服务器。

- 该 Sentinel 相信（believe）自己是客观领头 Sentinel 。

- 没有针对同一主服务器的故障转移操作在执行。

**Sentinel 规则 #14**\ ：

当故障转移操作发生，
并且以下条件为真时，
监视者 Sentinel 只负责进行日志记录，
以及在发布与订阅接口中产生相应的事件，
但并不会对任何实例进行重配置：

- 没有故障转移操作正在执行。

- 尽管一个被监视主服务器属下的从服务器转换成了主服务器，
  但只要这个主服务器的运行 ID 和原来的运行 ID 不同时，
  那么故障迁移就不会执行。
  因为运行 ID 的改变说明了从服务器只是因为重启而变成了主服务器，
  它不是被 Sentinel 选中的新主服务器。
  
**Sentinel 规则 #15**\ ：
领头 Sentinel 并不会立即开始故障转移操作。
它首先进入一个称为 ``wait-start`` 的状态，
这个状态会随机地维持 ``5`` 秒至 ``15`` 秒之间。
在这段时间内，
Sentinel 规则 #14 会生效：
如果在这段时间内，
有另一个从服务器被提升为主服务器，
那么这个领头 Sentinel 会放弃执行故障转移，
然后从领头 Sentinel 转变为观察者 Sentinel 。


完成故障转移
--------------------------------------------

当以下条件满足时，
Sentinel 将故障转移操作（process）视为完成：

- 被升级的从服务器（也即是新的主服务器）未处于主观下线状态。

- 从服务器升级为主服务器的过程已经完成。

- 旧主服务器属下的所有从服务器的复制目标已经改为新主服务器。
  注意，
  处于主观下线状态的从服务器会被忽略。

另外，
当以下任一条件被满足时，
故障转移操作会被中止：

- 被升级的从服务器（也即是新的主服务器）处于主观下线状态。

- 已经有一个从服务器被提升为新的主服务器。

- 在故障转移操作的最近一个步骤执行之后，
  已经过了 ``failover-timeout`` 秒。

``failover-timeout`` 的值可以在 ``sentinel.conf`` 文件中设置，
每个不同的从服务器可以设置不同的 ``failover-timeout`` 。

注意，
当领头 Sentinel 因为超时而停止故障转移操作时，
它向所有等待重配置的从服务器发送一条 :ref:`SLAVEOF` 命令，
Sentinel 会期望从服务器能够接收到这条 :ref:`SLAVEOF` 命令，
并最终完成和新主服务器的同步。

**Sentinel 规则 #16**\ ：

当以下条件被满足时，
领头 Sentinel 或者观察者 Sentinel 将故障转移操作视为完成（compelte）：

- 一个从服务器被顺利升级为主服务器，
  并且 Sentinel 已经通过 :ref:`INFO` 命令确认了这一点。

  另外，
  旧主服务器所属的所有从服务器，
  已经开始复制新主服务器，
  并且 Sentinel 已经通过 :ref:`INFO` 命令确认了这一点。

- 已经有一个从服务器被成功提升为主服务器，
  但对从服务器的重配置过程在 ``failover-timeout`` 所限定的时间内没有任何进展。

  在这种情况下，
  领头 Sentinel 会向目前所有未重配置的从服务器发送 :ref:`SLAVEOF` 命令，
  并期望从服务器可以接收到这个命令，
  并最终完成和新主服务器的同步。

在以上两个条件中，
被升级的新主服务器必须是可达（reachable）的：
也即是，
该主服务器未处于主观下线状态，
不然的话，
故障转移操作就不算完成。


在进行故障转移的过程中，领头 Sentinel 失效
--------------------------------------------

在一个领头 Sentinel 仍未提升某个从服务器为主服务器之前，
如果这个领头 Sentinel 进入主观下线状态，
并且其他 Sentinel 发现了这一情况，
那么只要剩余的 Sentinel 数量满足执行故障转移所需的数量，
故障转移就会自动继续进行：
剩余的 Sentinel 会重新选出新的领头 Sentinel ，
然后由新的领头 Sentinel 来继续执行故障转移操作。

假设故障转移正在进行中，
某个从服务器已经被提升为主服务器，
并且可能有数个从服务器已经开始对新主服务器进行复制，
如果这时领头 Sentinel 在超过 ``failover-timeout`` 选项所设定时长的 25% 以上的时间里，
没有进行任何动作，
那么多个观察者 Sentinel 中的其中一个会被选举为新的客观领头 Sentinel ，
然后这个新的领头 Sentinel 接替之前的领头 Sentinel ，
继续进行故障转移。

注意，
多个 Sentinel 向等待重配置的从服务器发送相同的 :ref:`SLAVEOF` 并不会引起竞争条件：

- 如果从服务器已经开始复制新服务器，
  那么这个重复的 :ref:`SLAVEOF` 命令会被安全地忽略，
  不会产生任何副作用或者性能问题。

- 如果从服务器没有开始复制新服务器，
  那么复制就会正式开始。

为了确保从服务器可以正确地复制新主服务器，
在一个领头 Sentinel 失效，
而另一个新的领头 Sentinel 出现之后，
新的领头 Sentinel 再次向所有从服务器发送 :ref:`SLAVEOF` 命令是必须的。

**Sentinel 规则 #17**\ ：

对于故障转移过程中的观察者 Sentinel 来说，
如果以下条件被满足，
那么这个观察者 Sentinel 会被转换成新的领头 Sentinel ，
并接替之前的领头 Sentinel ，
继续执行故障转移操作：

- 故障转移操作正在进行中，并且这个 Sentinel 是一个观察者 Sentinel 。

- 这个观察者 Sentinel 被选举为新的领头 Sentinel 。

- ``failover-timeout`` 选项所设定的时间的 25% 已经过去，
  故障转移操作仍然没有任何进展。
   

在故障转移过程中，被提升的新主服务器失效
--------------------------------------------

如果被提升的新主服务器在故障转移的过程中进入主观下线状态，
那么 Sentinel 没有办法知道故障转移过程已经中止。

不过，
当新主服务器的下线时长超过 ``down-after-milliseconds`` 选项所设定的毫秒数的十倍之后，
领头 Sentinel 或者观察者 Sentinel 会中止这次故障转移操作，
然后重新开始监视原来的旧主服务器。
如果这时旧主服务器仍然处于失效状态的话，
那么 Sentinel 会使用新的从从服务器，
从新开始一次新的故障转移操作。

注意，
当出现这种情况时，
可能已经有某些从服务器已经复制了进入主观下线状态的新服务器。
因此，
当领头 Sentinel 中止一次故障转移操作时，
它会向所有从服务器 —— 不管已经重配置完还是未配置完的 —— 都发送一个 :ref:`SLAVEOF` 命令，
让它们重新将复制的目标指向原本的主服务器。

**Sentinel 规则 #18**\ ：

当满足以下条件时，
一个 Sentinel （无论是领头还是观察者）会将故障转移过程视为中止：

- 领头 Sentinel 已经选出了一个要被提升的从服务器，
  或者观察者已经识别到有一个从服务器被提升成了主服务器。

- 被提升的从服务器（新主服务器）进入主观下线状态的时间已经超过了 ``down-after-milliseconds`` 选项所设定时间的十倍。


主服务器恢复（resurrecting）
--------------------------------------------

在故障转移操作执行之后，
如果已下线的旧主服务器重新上线，
那么只要你使用的是 2.6.13 或以上版本的 Redis ，
Sentinel 就可以将旧主服务器重新作为新主服务器的从服务器来使用。

以下是这个恢复工作的详细步骤：

- 当一次故障转移开始之后，
  领头 Sentinel 和观察者 Sentinel 都会将旧主服务器添加到一个列表中，
  这个列表包含了新主服务器的所有从服务器，
  并且列表中的旧主服务器会带有一个特殊的 ``DEMOTE`` 标记
  （这个标志可以用 ``SENTINEL SLAVES`` 命令查看）。

- 一旦旧主服务器重新上线，
  那么 Sentinel 会使用 :ref:`INFO` 命令与它进行通讯，
  根据 :ref:`INFO` 命令的返回值，
  如果这个服务器的角色仍然是主服务器的话，
  那么 Sentinel 会向这个服务器发送一个 :ref:`SLAVEOF` 命令，
  让它成为新主服务器的从服务器。

  当这个旧主服务器顺利成为从服务器时，
  ``DEMOTE`` 标记就会被清除。

并没有一个专门负责将旧主服务器转变为从服务器的 Sentinel ，
所以这个动作可能会因为 Sentinel 失效而失败。
与此同时，
当一个实例带有 ``DEMOTE`` 标志时，
这个实例永远不会被提升为主服务器。

当 Sentinel 通过 :ref:`INFO` 命令的回复确认一个旧主服务器已经成功转变为从服务器时，
Sentinel 会产生一个 ``+slave`` 事件。

**Sentinel 规则 #19**\ ：一旦故障转移操作开始，领头 Sentinel 和观察者 Sentinel 都会将旧主服务器设置为新主服务器的从服务器，并为它添加一个 ``DEMOTE`` 标志。

**Sentinel 规则 #20**\ ：一旦 Sentinel 发现旧主服务器重新上线，并且 :ref:`INFO` 命令显示这个服务器仍然为主服务器时， Sentinel 就会通过 :ref:`SLAVEOF` 命令将这个旧主服务器转换成新主服务器的从服务器。

**Sentinel 规则 #21**\ ：一旦 Sentinel 从 :ref:`INFO` 命令的回复中确认旧主服务器已经变为从服务器，那么旧主服务器的 ``DEMOTE`` 标记就会被清除。


TILT 模式
--------------------------------------------

Redis Sentinel 严重依赖计算机的时间功能：
比如说，
为了判断一个实例是否可用，
Sentinel 会记录这个实例最后一次相应 :ref:`PING` 命令的时间，
并将这个时间和当前时间进行对比，
从而知道这个实例有多长时间没有和 Sentinel 进行任何成功通讯。

不过，
一旦计算机的时间功能出现故障，
或者计算机非常忙碌，
又或者进程因为某些原因而被阻塞时，
Sentinel 可能也会跟着出现故障。

TILT 模式是一种特殊的保护模式：
当 Sentinel 发现系统有些不对劲时，
Sentinel 就会进入 TILT 模式。

因为 Sentinel 的时间中断器默认每秒执行 10 次，
所以我们预期时间中断器的两次执行之间的间隔为 100 毫秒左右。
Sentinel 的做法是，
记录上一次时间中断器执行时的时间，
并将它和这一次时间中断器执行的时间进行对比：

- 如果两次调用时间之间的差距为负值，
  或者非常大（超过 2 秒钟），
  那么 Sentinel 进入 TILT 模式。

- 如果 Sentinel 已经进入 TILT 模式，
  那么 Sentinel 延迟退出 TILT 模式的时间。

当 Sentinel 进入 TILT 模式时，
它仍然会继续监视所有目标，
但是：

- 它不再执行任何操作，比如故障转移。

- 当有实例向这个 Sentinel 发送 ``SENTINEL is-master-down-by-addr`` 命令时，
  Sentinel 返回负值：
  因为这个 Sentinel 所进行的下线判断已经不再准确。

如果 TILT 可以正常维持 30 秒钟，
那么 Sentinel 退出 TILT 模式。


处理 ``-BUSY`` 状态
--------------------------------------------

.. warning:: 该功能尚未实现

当 Lua 脚本的运行时间超过指定时限时，
Redis 就会返回 ``-BUSY`` 错误。

当出现这种情况时，
Sentinel 在尝试执行故障转移操作之前，
会先向服务器发送一个 :ref:`SCRIPT_KILL` 命令，
如果服务器正在执行的是一个只读脚本的话，
那么这个脚本就会被杀死，
服务器就会回到正常状态。


附录 A：算法与实现
--------------------------------------------

移除重复 Sentinel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

当一个动作需要由多个 Sentinel 共同做出确定，
又或者一个动作需要指定数量的 Sentinel 存在时才能发生，
那么我们必须保证，
统计的每个 Sentinel 都是独一无二的，
而不重复统计了一个 Sentinel 多次。

系统使用以下方法来移除重复的 Sentinel ：
每当一个 Sentinel 向频道发送带有地址（包括 IP 地址和端口）和运行 ID 的信息时，
如果系统不能在监视主服务器的 Sentinel 表中发现和这些信息完全匹配的 Sentinel ，
那么表中原有的地址相同或者运行 ID 相同的 Sentinel 就会被移除，
然后新的 Sentinel 会被添加到表中。

举个例子，
当一个 Sentinel 实例重启之后，
这个 Sentinel 的运行 ID 就会和之前的运行 ID 不同，
而原来的拥有相同地址的 Sentinel 就会从表中被移除，
新的 Sentinel 就会被添加到表中。

挑选从服务器进行升级
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

如果一个主服务器有多个从服务器，
那么新主服务器将根据从服务器的优先级来决定，
优先级较低的从服务器会被优先挑选。

.. warning::

    从服务器的优先级是一个配置选项，
    目前尚未真正实现。

所有曾经和主服务器长时间断开连接的从服务器都不会进入新主服务器的候选名单中。

如果有多个从服务器拥有相同的优先级，
那么按照字典排序，
运行 ID 最小的那个从服务器会被选中。

.. warning::

    因为从服务器优先级功能尚未实现，
    所以目前的新主服务器挑选只根据对运行 ID 进行字典序排序的结果来进行。

**Sentinel 规则 #22**\ ：
领头 Sentinel 会从目前状态良好的从服务器中（参考 Sentinel 规则 #11），
挑选出一个优先级最低的从服务器进行升级，
让它成为新的主服务器。
如果有多个相同优先级的从服务器，
那么运行 ID 按字典序排序最小的那个从服务器被选中。


附录 B： Sentinel 用法五分钟极速教程
-----------------------------------------------------------

如果你想尝试 Redis 的 Sentinel 系统，可以试着执行以下步骤：

1. 从 github 仓库中复制 Redis 项目的 ``unstable`` 分支。

2. 用 ``make`` 命令编译项目。

3. 执行 ``redis-server`` 程序，
   运行一些 Redis 服务器实例。
   最少要有一个主服务器和一个从服务器。

4. 使用 ``redis-sentinel /path/to/sentinel.conf`` 命令启动三个 Sentinel 实例。

三个实例的配置文件的内容和以下类似：

::

    port 26379
    sentinel monitor mymaster 127.0.0.1 6379 2
    sentinel down-after-milliseconds mymaster 5000
    sentinel failover-timeout mymaster 900000
    sentinel can-failover mymaster yes
    sentinel parallel-syncs mymaster 1

第一个端口号可以是 ``26379`` ，
第二个 Sentinel 实例的端口号可以是 ``26380`` ，
第三个 Sentinel 实例的端口号可以是 ``26381`` ，
或者其他任何未被使用的端口号。

注意 ``down-after-milliseconds`` 选项的值为 ``5`` 秒，
这对于测试 Sentinel 是一个不错的值，
但并不适用于生产环境。

启动 Sentinel 之后，
你应该可以在 Sentinel 的日志输出中发现类似以下内容：

::

    [4747] 23 Jul 14:49:15.883 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
    [4747] 23 Jul 14:49:19.645 * +sentinel sentinel 127.0.0.1:26379 127.0.0.1 26379 @ mymaster 127.0.0.1 6379
    [4747] 23 Jul 14:49:21.659 * +sentinel sentinel 127.0.0.1:26381 127.0.0.1 26381 @ mymaster 127.0.0.1 6379

你还可以通过客户端与 Sentinel 进行通讯：

::

    redis-cli -p 26379 sentinel masters
    1)  1) "name"
        2) "mymaster"
        3) "ip"
        4) "127.0.0.1"
        5) "port"
        6) "6379"
        7) "runid"
        8) "66215809eede5c0fdd20680cfb3dbd3bdf70a6f8"
        9) "flags"
        10) "master"
        11) "pending-commands"
        12) "0"
        13) "last-ok-ping-reply"
        14) "515"
        15) "last-ping-reply"
        16) "515"
        17) "info-refresh"
        18) "5116"
        19) "num-slaves"
        20) "1"
        21) "num-other-sentinels"
        22) "2"
        23) "quorum"
        24) "2"

要观察 Sentinel 处理故障迁移的执行过程，
你可以随意关停你的从服务器，
比如使用 ``DEBUG SEGFAULT`` 命令让它崩溃。

.. warning::

    这个示例目前还处于编写状态，
    所以内容比较简单，
    将来会添加更多内容。
