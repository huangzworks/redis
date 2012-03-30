.. _rpoplpush:

RPOPLPUSH
===========

**RPOPLPUSH source destination**

命令 `RPOPLPUSH`_ 在一个原子时间内，执行以下两个动作：

- 将列表 ``source`` 中的最后一个元素(尾元素)弹出，并返回给客户端。
- 将 ``source`` 弹出的元素插入到列表 ``destination`` ，作为 ``destination`` 列表的的头元素。

举个例子，你有两个列表 ``source`` 和 ``destination`` ， ``source`` 列表有元素 ``a, b, c`` ， ``destination`` 列表有元素 ``x, y, z`` ，执行 ``RPOPLPUSH source destination`` 之后， ``source`` 列表包含元素 ``a, b`` ， ``destination`` 列表包含元素 ``c, x, y, z``  ，并且元素 ``c`` 会被返回给客户端。

如果 ``source`` 不存在，值 ``nil`` 被返回，并且不执行其他动作。

如果 ``source`` 和 ``destination`` 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转(rotation)操作。

**可用版本：**
    >= 1.2.0

**时间复杂度：**
    O(1)

**返回值：**
    被弹出的元素。

::


    # source 和 destination 不同

    redis> LRANGE alpha 0 -1         # 查看所有元素
    1) "a"
    2) "b"
    3) "c"
    4) "d"

    redis> RPOPLPUSH alpha reciver   # 执行一次 RPOPLPUSH 看看
    "d"

    redis> LRANGE alpha 0 -1 
    1) "a"
    2) "b"
    3) "c"

    redis> LRANGE reciver 0 -1
    1) "d"

    redis> RPOPLPUSH alpha reciver   # 再执行一次，证实 RPOP 和 LPUSH 的位置正确
    "c"

    redis> LRANGE alpha 0 -1
    1) "a"
    2) "b"

    redis> LRANGE reciver 0 -1
    1) "c"
    2) "d"

    
    # source 和 destination 相同

    redis> LRANGE number 0 -1
    1) "1"
    2) "2"
    3) "3"
    4) "4"

    redis> RPOPLPUSH number number
    "4"

    redis> LRANGE number 0 -1           # 4 被旋转到了表头
    1) "4"
    2) "1"
    3) "2"
    4) "3"

    redis> RPOPLPUSH number number
    "3"

    redis> LRANGE number 0 -1           # 这次是 3 被旋转到了表头
    1) "3"
    2) "4"
    3) "1"
    4) "2"

模式： 安全的队列
----------------------------

Redis的列表经常被用作队列(queue)，用于在不同程序之间有序地交换消息(message)。一个客户端通过 :ref:`LPUSH` 命令将消息放入队列中，而另一个客户端通过 :ref:`RPOP` 或者 :ref:`BRPOP` 命令取出队列中等待时间最长的消息。

不幸的是，上面的队列方法是『不安全』的，因为在这个过程中，一个客户端可能在取出一个消息之后崩溃，而未处理完的消息也就因此丢失。

使用 `RPOPLPUSH`_ 命令(或者它的阻塞版本 :ref:`BRPOPLPUSH` )可以解决这个问题：因为它不仅返回一个消息，同时还将这个消息添加到另一个备份列表当中，如果一切正常的话，当一个客户端完成某个消息的处理之后，可以用 :ref:`LREM` 命令将这个消息从备份表删除。

最后，还可以添加一个客户端专门用于监视备份表，它自动地将超过一定处理时限的消息重新放入队列中去(负责处理该消息的客户端可能已经崩溃)，这样就不会丢失任何消息了。

模式：循环列表
--------------------

通过使用相同的 ``key`` 作为 `RPOPLPUSH`_ 命令的两个参数，客户端可以用一个接一个地获取列表元素的方式，取得列表的所有元素，而不必像 :ref:`LRANGE` 命令那样一下子将所有列表元素都从服务器传送到客户端中(两种方式的总复杂度都是 O(N))。

以上的模式甚至在以下的两个情况下也能正常工作：

- 有多个客户端同时对同一个列表进行旋转(rotating)，它们获取不同的元素，直到所有元素都被读取完，之后又从头开始。
- 有客户端在向列表尾部(右边)添加新元素。 

这个模式使得我们可以很容易实现这样一类系统：有 N 个客户端，需要连续不断地对一些元素进行处理，而且处理的过程必须尽可能地快。一个典型的例子就是服务器的监控程序：它们需要在尽可能短的时间内，并行地检查一组网站，确保它们的可访问性。

注意，使用这个模式的客户端是易于扩展(scala)且安全(reliable)的，因为就算接收到元素的客户端失败，元素还是保存在列表里面，不会丢失，等到下个迭代来临的时候，别的客户端又可以继续处理这些元素了。
