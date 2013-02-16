.. _sort:

SORT
=====

**SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]**

返回或保存给定列表、集合、有序集合 ``key`` 中经过排序的元素。

排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。


一般 SORT 用法
---------------------

最简单的 `SORT`_ 使用方法是 ``SORT key`` 和 ``SORT key DESC`` ：

- ``SORT key`` 返回键值从小到大排序的结果。

- ``SORT key DESC`` 返回键值从大到小排序的结果。

假设 ``today_cost`` 列表保存了今日的开销金额，
那么可以用 `SORT`_ 命令对它进行排序：

::

    # 开销金额列表

    redis> LPUSH today_cost 30 1.5 10 8
    (integer) 4

    # 排序

    redis> SORT today_cost 
    1) "1.5"
    2) "8"
    3) "10"
    4) "30"

    # 逆序排序

    redis 127.0.0.1:6379> SORT today_cost DESC
    1) "30"
    2) "10"
    3) "8"
    4) "1.5"


使用 ALPHA 修饰符对字符串进行排序
-------------------------------------

因为 `SORT`_ 命令默认排序对象为数字，
当需要对字符串进行排序时，
需要显式地在 `SORT`_ 命令之后添加 ``ALPHA`` 修饰符：

:: 

    # 网址

    redis> LPUSH website "www.reddit.com"
    (integer) 1

    redis> LPUSH website "www.slashdot.com"
    (integer) 2

    redis> LPUSH website "www.infoq.com"
    (integer) 3

    # 默认（按数字）排序

    redis> SORT website
    1) "www.infoq.com"
    2) "www.slashdot.com"
    3) "www.reddit.com"

    # 按字符排序

    redis> SORT website ALPHA
    1) "www.infoq.com"
    2) "www.reddit.com"
    3) "www.slashdot.com"

如果系统正确地设置了 ``LC_COLLATE`` 环境变量的话，Redis能识别 ``UTF-8`` 编码。


使用 LIMIT 修饰符限制返回结果
---------------------------------

排序之后返回元素的数量可以通过 ``LIMIT`` 修饰符进行限制，
修饰符接受 ``offset`` 和 ``count`` 两个参数：

- ``offset`` 指定要跳过的元素数量。
- ``count`` 指定跳过 ``offset`` 个指定的元素之后，要返回多少个对象。

以下例子返回排序结果的前 5 个对象( ``offset`` 为 ``0`` 表示没有元素被跳过)。

::

    # 添加测试数据，列表值为 1 指 10

    redis 127.0.0.1:6379> RPUSH rank 1 3 5 7 9
    (integer) 5

    redis 127.0.0.1:6379> RPUSH rank 2 4 6 8 10
    (integer) 10

    # 返回列表中最小的 5 个值

    redis 127.0.0.1:6379> SORT rank LIMIT 0 5
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"

可以组合使用多个修饰符。以下例子返回从大到小排序的前 5 个对象。

:: 

    redis 127.0.0.1:6379> SORT rank LIMIT 0 5 DESC
    1) "10"
    2) "9"
    3) "8"
    4) "7"
    5) "6"

使用外部 key 进行排序
---------------------------

可以使用外部 ``key`` 的数据作为权重，代替默认的直接对比键值的方式来进行排序。

假设现在有用户数据如下：

.. include:: _user_info_table.include

以下代码将数据输入到 Redis 中：

::

    # admin

    redis 127.0.0.1:6379> LPUSH uid 1
    (integer) 1

    redis 127.0.0.1:6379> SET user_name_1 admin
    OK

    redis 127.0.0.1:6379> SET user_level_1 9999
    OK

    # jack

    redis 127.0.0.1:6379> LPUSH uid 2
    (integer) 2

    redis 127.0.0.1:6379> SET user_name_2 jack
    OK

    redis 127.0.0.1:6379> SET user_level_2 10
    OK

    # peter

    redis 127.0.0.1:6379> LPUSH uid 3
    (integer) 3

    redis 127.0.0.1:6379> SET user_name_3 peter
    OK

    redis 127.0.0.1:6379> SET user_level_3 25
    OK

    # mary

    redis 127.0.0.1:6379> LPUSH uid 4
    (integer) 4

    redis 127.0.0.1:6379> SET user_name_4 mary
    OK

    redis 127.0.0.1:6379> SET user_level_4 70
    OK

BY 选项
^^^^^^^^^^^

默认情况下， ``SORT uid`` 直接按 ``uid`` 中的值排序：

::

    redis 127.0.0.1:6379> SORT uid
    1) "1"      # admin 
    2) "2"      # jack
    3) "3"      # peter
    4) "4"      # mary

通过使用 ``BY`` 选项，可以让 ``uid`` 按其他键的元素来排序。

比如说，
以下代码让 ``uid`` 键按照 ``user_level_{uid}`` 的大小来排序：

::

    redis 127.0.0.1:6379> SORT uid BY user_level_*
    1) "2"      # jack , level = 10 
    2) "3"      # peter, level = 25
    3) "4"      # mary, level = 70
    4) "1"      # admin, level = 9999

``user_level_*`` 是一个占位符，
它先取出 ``uid`` 中的值，
然后再用这个值来查找相应的键。

比如在对 ``uid`` 列表进行排序时，
程序就会先取出 ``uid`` 的值 ``1`` 、 ``2`` 、 ``3`` 、 ``4`` ，
然后使用 ``user_level_1`` 、 ``user_level_2`` 、 ``user_level_3`` 和 ``user_level_4`` 的值作为排序 ``uid`` 的权重。

GET 选项
^^^^^^^^^^^

使用 ``GET`` 选项，
可以根据排序的结果来取出相应的键值。

比如说，
以下代码先排序 ``uid`` ，
再取出键 ``user_name_{uid}`` 的值：

::

    redis 127.0.0.1:6379> SORT uid GET user_name_*
    1) "admin"
    2) "jack"
    3) "peter"
    4) "mary"

组合使用 BY 和 GET
^^^^^^^^^^^^^^^^^^^^^^^^

通过组合使用 ``BY`` 和 ``GET`` ，
可以让排序结果以更直观的方式显示出来。

比如说，
以下代码先按 ``user_level_{uid}`` 来排序 ``uid`` 列表，
再取出相应的 ``user_name_{uid}`` 的值：

::

    redis 127.0.0.1:6379> SORT uid BY user_level_* GET user_name_*
    1) "jack"       # level = 10
    2) "peter"      # level = 25
    3) "mary"       # level = 70
    4) "admin"      # level = 9999

现在的排序结果要比只使用 ``SORT uid BY user_level_*`` 要直观得多。

获取多个外部键
^^^^^^^^^^^^^^^^^

可以同时使用多个 ``GET`` 选项，
获取多个外部键的值。

以下代码就按 ``uid`` 分别获取 ``user_level_{uid}`` 和 ``user_name_{uid}`` ：

::

    redis 127.0.0.1:6379> SORT uid GET user_level_* GET user_name_*
    1) "9999"       # level
    2) "admin"      # name
    3) "10"
    4) "jack"
    5) "25"
    6) "peter"
    7) "70"
    8) "mary"

``GET`` 有一个额外的参数规则，那就是 —— 可以用 ``#`` 获取被排序键的值。

以下代码就将 ``uid`` 的值、及其相应的 ``user_level_*`` 和 ``user_name_*`` 都返回为结果：

::

    redis 127.0.0.1:6379> SORT uid GET # GET user_level_* GET user_name_*
    1) "1"          # uid
    2) "9999"       # level
    3) "admin"      # name
    4) "2"
    5) "10"
    6) "jack"
    7) "3"
    8) "25"
    9) "peter"
    10) "4"
    11) "70"
    12) "mary"

获取外部键，但不进行排序
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

通过将一个不存在的键作为参数传给 ``BY`` 选项，
可以让 ``SORT`` 跳过排序操作，
直接返回结果：

::

    redis 127.0.0.1:6379> SORT uid BY not-exists-key
    1) "4"
    2) "3"
    3) "2"
    4) "1"

这种用法在单独使用时，没什么实际用处。

不过，通过将这种用法和 ``GET`` 选项配合，
就可以在不排序的情况下，
获取多个外部键，
相当于执行一个整合的获取操作（类似于 SQL 数据库的 ``join`` 关键字）。

以下代码演示了，如何在不引起排序的情况下，使用 ``SORT`` 、 ``BY`` 和 ``GET`` 获取多个外部键：

::

    redis 127.0.0.1:6379> SORT uid BY not-exists-key GET # GET user_level_* GET user_name_*
    1) "4"      # id
    2) "70"     # level
    3) "mary"   # name
    4) "3"
    5) "25"
    6) "peter"
    7) "2"
    8) "10"
    9) "jack"
    10) "1"
    11) "9999"
    12) "admin"

将哈希表作为 GET 或 BY 的参数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

除了可以将字符串键之外，
哈希表也可以作为 ``GET`` 或 ``BY`` 选项的参数来使用。

比如说，对于前面给出的用户信息表：

.. include:: _user_info_table.include

我们可以不将用户的名字和级别保存在 ``user_name_{uid}`` 和 ``user_level_{uid}`` 两个字符串键中，
而是用一个带有 ``name`` 域和 ``level`` 域的哈希表 ``user_info_{uid}`` 来保存用户的名字和级别信息：

::

    redis 127.0.0.1:6379> HMSET user_info_1 name admin level 9999
    OK

    redis 127.0.0.1:6379> HMSET user_info_2 name jack level 10
    OK

    redis 127.0.0.1:6379> HMSET user_info_3 name peter level 25
    OK

    redis 127.0.0.1:6379> HMSET user_info_4 name mary level 70
    OK

之后， ``BY`` 和 ``GET`` 选项都可以用 ``key->field`` 的格式来获取哈希表中的域的值，
其中 ``key`` 表示哈希表键，
而 ``field`` 则表示哈希表的域：

::

    redis 127.0.0.1:6379> SORT uid BY user_info_*->level
    1) "2"
    2) "3"
    3) "4"
    4) "1"

    redis 127.0.0.1:6379> SORT uid BY user_info_*->level GET user_info_*->name
    1) "jack"
    2) "peter"
    3) "mary"
    4) "admin"


保存排序结果
----------------

默认情况下， `SORT`_ 操作只是简单地返回排序结果，并不进行任何保存操作。

通过给 ``STORE`` 选项指定一个 ``key`` 参数，可以将排序结果保存到给定的键上。

如果被指定的 ``key`` 已存在，那么原有的值将被排序结果覆盖。

::

    # 测试数据

    redis 127.0.0.1:6379> RPUSH numbers 1 3 5 7 9
    (integer) 5

    redis 127.0.0.1:6379> RPUSH numbers 2 4 6 8 10
    (integer) 10

    redis 127.0.0.1:6379> LRANGE numbers 0 -1
    1) "1"
    2) "3"
    3) "5"
    4) "7"
    5) "9"
    6) "2"
    7) "4"
    8) "6"
    9) "8"
    10) "10"
    
    redis 127.0.0.1:6379> SORT numbers STORE sorted-numbers
    (integer) 10

    # 排序后的结果

    redis 127.0.0.1:6379> LRANGE sorted-numbers 0 -1
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    6) "6"
    7) "7"
    8) "8"
    9) "9"
    10) "10"

可以通过将 `SORT`_ 命令的执行结果保存，并用 :ref:`EXPIRE` 为结果设置生存时间，以此来产生一个 `SORT`_ 操作的结果缓存。

这样就可以避免对 `SORT`_ 操作的频繁调用：只有当结果集过期时，才需要再调用一次 `SORT`_ 操作。

另外，为了正确实现这一用法，你可能需要加锁以避免多个客户端同时进行缓存重建(也就是多个客户端，同一时间进行 `SORT`_ 操作，并保存为结果集)，具体参见 :ref:`setnx` 命令。


**可用版本：**
    >= 1.0.0

**时间复杂度：**
    | O(N+M*log(M))， ``N`` 为要排序的列表或集合内的元素数量， ``M`` 为要返回的元素数量。
    | 如果只是使用 `SORT`_ 命令的 ``GET`` 选项获取数据而没有进行排序，时间复杂度 O(N)。
                               
**返回值：**
    | 没有使用 ``STORE`` 参数，返回列表形式的排序结果。
    | 使用 ``STORE`` 参数，返回排序结果的元素数量。
