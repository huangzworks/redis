.. _unwatch:

UNWATCH
========

**UNWATCH**

取消 `WATCH`_ 命令对所有 key 的监视。

如果在执行 `WATCH`_ 命令之后， `EXEC`_ 命令或 `DISCARD`_ 命令先被执行了的话，那么就不需要再执行 `UNWATCH`_ 了。

因为 `EXEC`_ 命令会执行事务，因此 `WATCH`_ 命令的效果已经产生了；而 `DISCARD`_ 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 `UNWATCH`_ 了。

**可用版本：**
    >= 2.2.0

**时间复杂度：**
    O(1)

**返回值：**
    总是 ``OK`` 。

::

    redis> WATCH lock lock_times
    OK

    redis> UNWATCH
    OK
