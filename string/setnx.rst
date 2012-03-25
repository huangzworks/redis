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



