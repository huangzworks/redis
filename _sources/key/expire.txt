.. _expire:

EXPIRE
=======

**EXPIRE key seconds**

为给定 ``key`` 设置生存时间，当 ``key`` 过期时(生存时间为 ``0`` )，它会被自动删除。

在 Redis 中，带有生存时间的 ``key`` 被称为『易失的』(volatile)。

生存时间可以通过使用 :ref:`DEL` 命令来删除整个 ``key`` 来移除，或者被 :ref:`SET` 和 :ref:`GETSET` 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 ``key`` 的值而不是用一个新的 ``key`` 值来代替(replace)它的话，那么生存时间不会被改变。

比如说，对一个 ``key`` 执行 :ref:`INCR` 命令，对一个列表进行 :ref:`LPUSH` 命令，或者对一个哈希表执行 :ref:`HSET` 命令，这类操作都不会修改 ``key`` 本身的生存时间。

另一方面，如果使用 :doc:`rename` 对一个 ``key`` 进行改名，那么改名后的 ``key`` 的生存时间和改名前一样。

:doc:`rename` 命令的另一种可能是，尝试将一个带生存时间的 ``key`` 改名成另一个带生存时间的 ``another_key`` ，这时旧的 ``another_key`` (以及它的生存时间)会被删除，然后旧的 ``key`` 会改名为 ``another_key`` ，因此，新的 ``another_key`` 的生存时间也和原本的 ``key`` 一样。

使用 :doc:`persist` 命令可以在不删除 ``key`` 的情况下，移除 ``key`` 的生存时间，让 ``key`` 重新成为一个『持久的』(persistent) ``key`` 。

**更新生存时间**

可以对一个已经带有生存时间的 ``key`` 执行 :ref:`EXPIRE` 命令，新指定的生存时间会取代旧的生存时间。

**过期时间的精确度**

在 Redis 2.4 版本中，过期时间的延迟在 1 秒钟之内 —— 也即是，就算 ``key`` 已经过期，但它还是可能在过期之后一秒钟之内被访问到，而在新的 Redis 2.6 版本中，延迟被降低到 1 毫秒之内。

**Redis 2.1.3 之前的不同之处**

在 Redis 2.1.3 之前的版本中，修改一个带有生存时间的 ``key`` 会导致整个 ``key`` 被删除，这一行为是受当时复制(replication)层的限制而作出的，现在这一限制已经被修复。

**可用版本：**
    >=  1.0.0

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功返回 ``1`` 。
    | 当 ``key`` 不存在或者不能为 ``key`` 设置生存时间时(比如在低于 2.1.3 版本的 Redis 中你尝试更新 ``key`` 的生存时间)，返回 ``0`` 。

::

    redis> SET cache_page "www.google.com"
    OK

    redis> EXPIRE cache_page 30  # 设置过期时间为 30 秒
    (integer) 1

    redis> TTL cache_page    # 查看剩余生存时间
    (integer) 23

    redis> EXPIRE cache_page 30000   # 更新过期时间
    (integer) 1

    redis> TTL cache_page
    (integer) 29996

模式：导航会话
-----------------

假设你有一项 web 服务，打算根据用户最近访问的 N 个页面来进行物品推荐，并且假设用户停止阅览超过 60 秒，那么就清空阅览记录(为了减少物品推荐的计算量，并且保持推荐物品的新鲜度)。

这些最近访问的页面记录，我们称之为『导航会话』(Navigation session)，可以用 :ref:`INCR` 和 :ref:`RPUSH` 命令在 Redis 中实现它：每当用户阅览一个网页的时候，执行以下代码：

::
    
    MULTI
        RPUSH pagewviews.user:<userid> http://.....
        EXPIRE pagewviews.user:<userid> 60
    EXEC

如果用户停止阅览超过 60 秒，那么它的导航会话就会被清空，当用户重新开始阅览的时候，系统又会重新记录导航会话，继续进行物品推荐。


