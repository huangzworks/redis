.. _object:

OBJECT
======

**OBJECT subcommand [arguments [arguments]]**

`OBJECT`_ 命令允许从内部察看给定 ``key`` 的 Redis 对象。

| 它通常用在除错(debugging)或者了解为了节省空间而对 ``key`` 使用特殊编码的情况。
| 当将Redis用作缓存程序时，你也可以通过 `OBJECT`_ 命令中的信息，决定 ``key`` 的驱逐策略(eviction policies)。

OBJECT 命令有多个子命令：

*  ``OBJECT REFCOUNT <key>`` 返回给定 ``key`` 引用所储存的值的次数。此命令主要用于除错。
*  ``OBJECT ENCODING <key>`` 返回给定 ``key`` 锁储存的值所使用的内部表示(representation)。
*  ``OBJECT IDLETIME <key>`` 返回给定 ``key`` 自储存以来的空闲时间(idle， 没有被读取也没有被写入)，以秒为单位。

| 对象可以以多种方式编码：

* 字符串可以被编码为 ``raw`` (一般字符串)或 ``int`` (为了节约内存，Redis 会将字符串表示的 64 位有符号整数编码为整数来进行储存）。
* 列表可以被编码为 ``ziplist`` 或 ``linkedlist`` 。 ``ziplist`` 是为节约大小较小的列表空间而作的特殊表示。
* 集合可以被编码为 ``intset`` 或者 ``hashtable`` 。 ``intset`` 是只储存数字的小集合的特殊表示。
* 哈希表可以编码为 ``zipmap`` 或者 ``hashtable`` 。 ``zipmap`` 是小哈希表的特殊表示。
* 有序集合可以被编码为 ``ziplist`` 或者 ``skiplist`` 格式。 ``ziplist`` 用于表示小的有序集合，而 ``skiplist`` 则用于表示任何大小的有序集合。

| 假如你做了什么让 Redis 没办法再使用节省空间的编码时(比如将一个只有 1 个元素的集合扩展为一个有 100 万个元素的集合)，特殊编码类型(specially encoded types)会自动转换成通用类型(general type)。

**可用版本：**
    >= 2.2.3

**时间复杂度：**
    O(1)

**返回值：**
    |  ``REFCOUNT`` 和 ``IDLETIME`` 返回数字。
    |  ``ENCODING`` 返回相应的编码类型。

::

    redis> SET game "COD"           # 设置一个字符串
    OK
    
    redis> OBJECT REFCOUNT game     # 只有一个引用
    (integer) 1
    
    redis> OBJECT IDLETIME game     # 等待一阵。。。然后查看空闲时间
    (integer) 90
    
    redis> GET game                 # 提取game， 让它处于活跃(active)状态
    "COD"

    redis> OBJECT IDLETIME game     # 不再处于空闲状态
    (integer) 0

    redis> OBJECT ENCODING game     # 字符串的编码方式
    "raw"

    redis> SET big-number 23102930128301091820391092019203810281029831092  # 非常长的数字会被编码为字符串
    OK

    redis> OBJECT ENCODING big-number
    "raw"

    redis> SET small-number 12345  # 而短的数字则会被编码为整数
    OK

    redis> OBJECT ENCODING small-number
    "int"

