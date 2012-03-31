.. _unwatch:

UNWATCH
========

**UNWATCH**

取消 :ref:`WATCH` 命令对所有 key 的监视。

如果在执行 :ref:`WATCH` 命令之后， :ref:`EXEC` 命令或 :ref:`DISCARD` 命令先被执行了的话，那么就不需要再执行 `UNWATCH`_ 了。

因为 :ref:`EXEC` 命令会执行事务，因此 :ref:`WATCH` 命令的效果已经产生了；而 :ref:`DISCARD` 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 `UNWATCH`_ 了。

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
