.. _config_rewrite:

CONFIG REWRITE
===================

**CONFIG REWRITE**

:ref:`CONFIG_REWRITE` 命令对启动 Redis 服务器时所指定的 ``redis.conf`` 文件进行改写：
因为 :ref:`CONFIG_SET` 命令可以对服务器的当前配置进行修改，
而修改后的配置可能和 ``redis.conf`` 文件中所描述的配置不一样，
:ref:`CONFIG_REWRITE` 的作用就是通过尽可能少的修改，
将服务器当前所使用的配置记录到 ``redis.conf`` 文件中。

重写会以非常保守的方式进行：

- 原有 ``redis.conf`` 文件的整体结构和注释会被尽可能地保留。

- 如果一个选项已经存在于原有 ``redis.conf`` 文件中 ，
  那么对该选项的重写会在选项原本所在的位置（行号）上进行。

- 如果一个选项不存在于原有 ``redis.conf`` 文件中，
  并且该选项被设置为默认值，
  那么重写程序不会将这个选项添加到重写后的 ``redis.conf`` 文件中。

- 如果一个选项不存在于原有 ``redis.conf`` 文件中，
  并且该选项被设置为非默认值，
  那么这个选项将被添加到重写后的 ``redis.conf`` 文件的末尾。

- 未使用的行会被留白。
  比如说，
  如果你在原有 ``redis.conf`` 文件上设置了数个关于 ``save`` 选项的参数，
  但现在你将这些 ``save`` 参数的一个或全部都关闭了，
  那么这些不再使用的参数原本所在的行就会变成空白的。

即使启动服务器时所指定的 ``redis.conf`` 文件已经不再存在，
:ref:`CONFIG_REWRITE` 命令也可以重新构建并生成出一个新的 ``redis.conf`` 文件。

另一方面，
如果启动服务器时没有载入 ``redis.conf`` 文件，
那么执行 :ref:`CONFIG_REWRITE` 命令将引发一个错误。


原子性重写
------------------------

对 ``redis.conf`` 文件的重写是原子性的，
并且是一致的：
如果重写出错或重写期间服务器崩溃，
那么重写失败，
原有 ``redis.conf`` 文件不会被修改。
如果重写成功，
那么 ``redis.conf`` 文件为重写后的新文件。


可用版本
-----------------------

>= 2.8.0


返回值
----------------------

一个状态值：如果配置重写成功则返回 ``OK`` ，失败则返回一个错误。


测试
----------------------

以下是执行 :ref:`CONFIG_REWRITE` 前，
被载入到 Redis 服务器的 ``redis.conf`` 文件中关于 ``appendonly`` 选项的设置：

::

    # ... 其他选项

    appendonly no

    # ... 其他选项

在执行以下命令之后：

::

    127.0.0.1:6379> CONFIG GET appendonly           # appendonly 处于关闭状态
    1) "appendonly"
    2) "no"

    127.0.0.1:6379> CONFIG SET appendonly yes       # 打开 appendonly
    OK

    127.0.0.1:6379> CONFIG GET appendonly
    1) "appendonly"
    2) "yes"

    127.0.0.1:6379> CONFIG REWRITE                  # 将 appendonly 的修改写入到 redis.conf 中
    OK

重写后的 ``redis.conf`` 文件中的 ``appendonly`` 选项将被改写：

::

    # ... 其他选项

    appendonly yes

    # ... 其他选项
