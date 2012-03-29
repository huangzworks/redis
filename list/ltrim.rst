.. _ltrim:

LTRIM
=======

**LTRIM key start stop**

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

举个例子，执行命令\ ``LTRIM list 0 2``\ ，表示只保留列表\ ``list``\ 的前三个元素，其余元素全部删除。

下标(index)参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 表示列表的第一个元素，以\ ``1``\ 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以\ ``-1``\ 表示列表的最后一个元素，\ ``-2``\ 表示列表的倒数第二个元素，以此类推。

当\ ``key``\ 不是列表类型时，返回一个错误。

\ `LTRIM`_\ 命令通常和\ `LPUSH`_\ 命令或\ `RPUSH`_\ 命令配合使用，举个例子：

::

    LPUSH log newest_log
    LTRIM log 0 99

这个例子模拟了一个日志程序，每次将最新日志\ ``newest_log``\ 放到\ ``log``\ 列表中，并且只保留最新的\ ``100``\ 项。注意当这样使用\ ``LTRIM``\ 命令时，时间复杂度是O(1)，因为平均情况下，每次只有一个元素被移除。

**注意LTRIM命令和编程语言区间函数的区别**

假如你有一个包含一百个元素的列表\ ``list``\ ，对该列表执行\ ``LTRIM list 0 10``\ ，结果是一个包含11个元素的列表，这表明\ ``stop``\ 下标也在\ `LTRIM`_\ 命令的取值范围之内(闭区间)，这和某些语言的区间函数可能不一致，比如Ruby的\ ``Range.new``\ 、\ ``Array#slice``\ 和Python的\ ``range()``\ 函数。

**超出范围的下标**

超出范围的下标值不会引起错误。

如果\ ``start``\ 下标比列表的最大下标\ ``end``\ (\ ``LLEN list``\ 减去\ ``1``\ )还要大，或者\ ``start > stop``\ ，\ `LTRIM`_\ 返回一个空列表(因为\ `LTRIM`_\ 已经将整个列表清空)。

如果\ ``stop``\ 下标比\ ``end``\ 下标还要大，Redis将\ ``stop``\ 的值设置为\ ``end``\ 。

**时间复杂度:**
    O(N)，\ ``N``\ 为被移除的元素的数量。

**返回值:**
    | 命令执行成功时，返回\ ``ok``\ 。

::

    # 情况1：一般情况下标

    redis> LRANGE alpha 0 -1 # 建立一个 5 元素的列表
    1) "h"
    2) "e"
    3) "l"
    4) "l"
    5) "o"

    redis> LTRIM alpha 1 -1  # 删除索引为 0 的元素
    OK

    redis> LRANGE alpha 0 -1 # "h" 被删除
    1) "e"
    2) "l"
    3) "l"
    4) "o"

    
    # 情况2：stop 下标比元素的最大下标要大

    redis> LTRIM alpha 1 10086 
    OK
    redis> LRANGE alpha 0 -1
    1) "l"
    2) "l"
    3) "o"

    
    # 情况3：start和stop下标都比最大下标要大，且start < sotp

    redis> LTRIM alpha 10086 200000  
    OK
    redis> LRANGE alpha 0 -1 # 整个列表被清空，等同于 DEL alpha
    (empty list or set)


    # 情况4：start > stop

    redis> LRANGE alpha 0 -1 # 在新建一个列表
    1) "h"
    2) "u"
    3) "a"
    4) "n"
    5) "g"
    6) "z"

    redis> LTRIM alpha 10086 4
    OK

    redis> LRANGE alpha 0 -1 # 列表同样被清空
    (empty list or set)


