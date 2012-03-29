.. _script_kill:

SCRIPT KILL
=============

**SCRIPT KILL**

杀死当前正在运行的 Lua 脚本，当且仅当这个脚本没有执行过任何写操作时，这个命令才生效。

这个命令主要用于终止运行时间过长的脚本，比如一个因为 BUG 而发生无限 loop 的脚本，诸如此类。

`SCRIPT KILL`_ 执行之后，当前正在运行的脚本会被杀死，执行这个脚本的客户端会从 :ref:`EVAL` 命令的阻塞当中退出，并收到一个错误作为返回值。

另一方面，假如当前正在运行的脚本已经执行过写操作，那么即使执行 `SCRIPT KILL`_ ，也无法将它杀死，因为这是违反 Lua 脚本的原子性执行原则的。在这种情况下，唯一可行的办法是使用 ``SHUTDOWN NOSAVE`` 命令，通过停止整个 Redis 进程来停止脚本的运行，并防止不完整(half-written)的信息被写入数据库中。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 :ref:`EVAL` 命令。

**可用版本：**
    >= 2.6.0

**时间复杂度：**
    O(1)

**返回值：**
    执行成功返回 ``OK`` ，否则返回一个错误。

::
    
    # 没有脚本在执行时

    redis> SCRIPT KILL
    (error) ERR No scripts in execution right now.

    # 成功杀死脚本时

    redis> SCRIPT KILL
    OK
    (1.30s)

    # 尝试杀死一个已经执行过写操作的脚本，失败

    redis> SCRIPT KILL
    (error) ERR Sorry the script already executed write commands against the dataset. You can either wait the script termination or kill the server in an hard way using the SHUTDOWN NOSAVE command.
    (1.69s)

以下是脚本被杀死之后，返回给执行脚本的客户端的错误：

::

    redis> EVAL "while true do end" 0
    (error) ERR Error running script (call to f_694a5fe1ddb97a4c6a1bf299d9537c7d3d0f84e7): Script killed by user with SCRIPT KILL... 
    (5.00s)

