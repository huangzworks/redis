.. _sort:

SORT
=====

**SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]**

返回或保存给定列表、集合、有序集合 ``key`` 中经过排序的元素。

排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。

**一般SORT用法**

最简单的 `SORT`_ 使用方法是 ``SORT key`` 。

假设 ``today_cost`` 是一个保存数字的列表， `SORT`_ 命令默认会返回该列表值的递增(从小到大)排序结果。

::

    # 将数据一一加入到列表中

    redis> LPUSH today_cost 30
    (integer) 1

    redis> LPUSH today_cost 1.5
    (integer) 2

    redis> LPUSH today_cost 10
    (integer) 3

    redis> LPUSH today_cost 8
    (integer) 4

    # 排序

    redis> SORT today_cost 
    1) "1.5"
    2) "8"
    3) "10"
    4) "30"

当数据集中保存的是字符串值时，你可以用 ``ALPHA`` 修饰符(modifier)进行排序。
   
:: 

    # 将数据一一加入到列表中

    redis> LPUSH website "www.reddit.com"
    (integer) 1
    redis> LPUSH website "www.slashdot.com"
    (integer) 2
    redis> LPUSH website "www.infoq.com"
    (integer) 3

    # 默认排序

    redis> SORT website
    1) "www.infoq.com"
    2) "www.slashdot.com"
    3) "www.reddit.com"

    # 按字符排序

    redis> SORT website ALPHA
    1) "www.infoq.com"
    2) "www.reddit.com"
    3) "www.slashdot.com"

如果你正确设置了 ``!LC_COLLATE`` 环境变量的话，Redis能识别 ``UTF-8`` 编码。

| 排序之后返回的元素数量可以通过 ``LIMIT`` 修饰符进行限制。
|  ``LIMIT`` 修饰符接受两个参数： ``offset`` 和 ``count`` 。
|  ``offset`` 指定要跳过的元素数量， ``count`` 指定跳过 ``offset`` 个指定的元素之后，要返回多少个对象。

以下例子返回排序结果的前5个对象( ``offset`` 为 ``0`` 表示没有元素被跳过)。

::

    # 将数据一一加入到列表中

    redis> LPUSH rank 30
    (integer) 1
    redis> LPUSH rank 56
    (integer) 2
    redis> LPUSH rank 42
    (integer) 3
    redis> LPUSH rank 22
    (integer) 4
    redis> LPUSH rank 0
    (integer) 5
    redis> LPUSH rank 11
    (integer) 6
    redis> LPUSH rank 32
    (integer) 7
    redis> LPUSH rank 67
    (integer) 8
    redis> LPUSH rank 50
    (integer) 9
    redis> LPUSH rank 44
    (integer) 10
    redis> LPUSH rank 55
    (integer) 11

    # 排序

    redis> SORT rank LIMIT 0 5  # 返回排名前五的元素
    1) "0"
    2) "11"
    3) "22"
    4) "30"
    5) "32"

修饰符可以组合使用。以下例子返回降序(从大到小)的前 5 个对象。

:: 

    redis> SORT rank LIMIT 0 5 DESC
    1) "78"
    2) "67"
    3) "56"
    4) "55"
    5) "50"

**使用外部key进行排序**

有时候你会希望使用外部的 ``key`` 作为权重来比较元素，代替默认的对比方法。

假设现在有用户(user)数据如下：

    =====  ====== ======
    id     name   level
    =====  ====== ======
    1      admin   9999
    2      huangz  10   
    59230  jack    3   
    222    hacker  9999 
    =====  ====== ======

|  ``id`` 数据保存在 ``key`` 名为 ``user_id`` 的列表中。
|  ``name`` 数据保存在 ``key`` 名为 ``user_name_{id}`` 的列表中
|  ``level`` 数据保存在 ``user_level_{id}`` 的 ``key`` 中。

::

    # 先将要使用的数据加入到数据库中

    # admin

    redis> LPUSH user_id 1
    (integer) 1
    redis> SET user_name_1 admin
    OK
    redis> SET user_level_1 9999
    OK

    # huangz

    redis> LPUSH user_id 2
    (integer) 2
    redis> SET user_name_2 huangz
    OK
    redis> SET user_level_2 10
    OK

    # jack

    redis> LPUSH user_id 59230
    (integer) 3
    redis> SET user_name_59230 jack
    OK
    redis> SET user_level_59230 3
    OK

    # hacker

    redis> LPUSH user_id 222
    (integer) 4
    redis> SET user_name_222 hacker
    OK
    redis> SET user_level_222 9999
    OK

如果希望按 ``level`` 从大到小排序 ``user_id`` ，可以使用以下命令：

::

    redis> SORT user_id BY user_level_* DESC
    1) "222"    # hacker
    2) "1"      # admin
    3) "2"      # huangz    
    4) "59230"  # jack

但是有时候只是返回相应的 ``id`` 没有什么用，你可能更希望排序后返回 ``id`` 对应的用户名，这样更友好一点，使用 ``GET`` 选项可以做到这一点：

::

    redis> SORT user_id BY user_level_* DESC GET user_name_*
    1) "hacker"
    2) "admin"
    3) "huangz"
    4) "jack"

可以多次地、有序地使用 ``GET`` 操作来获取更多外部 ``key`` 。

比如你不但希望获取用户名，还希望连用户的密码也一并列出，可以使用以下命令：

::

    # 先添加一些测试数据

    redis> SET user_password_222 "hey,im in"
    OK
    redis> SET user_password_1 "a_long_long_password"
    OK
    redis> SET user_password_2 "nobodyknows"
    OK
    redis> SET user_password_59230 "jack201022"
    OK

    # 获取name和password

    redis> SORT user_id BY user_level_* DESC GET user_name_* GET user_password_*
    1) "hacker"       # 用户名
    2) "hey,im in"    # 密码
    3) "jack"
    4) "jack201022"
    5) "huangz"
    6) "nobodyknows"
    7) "admin"
    8) "a_long_long_password"

    # 注意GET操作是有序的，GET user_name_* GET user_password_* 和 GET user_password_* GET user_name_*返回的结果位置不同

    redis> SORT user_id BY user_level_* DESC GET user_password_* GET user_name_*
    1) "hey,im in"    # 密码
    2) "hacker"       # 用户名
    3) "jack201022"
    4) "jack"
    5) "nobodyknows"
    6) "huangz"
    7) "a_long_long_password"
    8) "admin"

``GET`` 还有一个特殊的规则—— ``"GET #"`` ，用于获取被排序对象(我们这里的例子是 ``user_id`` )的当前元素。

比如你希望 ``user_id`` 按 ``level`` 排序，还要列出 ``id`` 、 ``name`` 和 ``password`` ，可以使用以下命令：

::

    redis> SORT user_id BY user_level_* DESC GET # GET user_name_* GET user_password_*
    1) "222"          # id
    2) "hacker"       # name
    3) "hey,im in"    # password
    4) "1"
    5) "admin"
    6) "a_long_long_password"
    7) "2"
    8) "huangz"
    9) "nobodyknows"
    10) "59230"
    11) "jack"
    12) "jack201022"

**只获取对象而不排序**
    
``BY`` 修饰符可以将一个不存在的 ``key`` 当作权重，让 `SORT`_ 跳过排序操作。

该方法用于你希望获取外部对象而又不希望引起排序开销时使用。

::

    # 确保fake_key不存在

    redis> EXISTS fake_key
    (integer) 0

    # 以fake_key作BY参数，不排序，只GET name 和 GET password

    redis> SORT user_id BY fake_key GET # GET user_name_* GET user_password_*
    1) "222"        # id
    2) "hacker"     # user_name
    3) "hey,im in"  # password
    4) "59230"
    5) "jack"
    6) "jack201022"
    7) "2"
    8) "huangz"
    9) "nobodyknows"
    10) "1"
    11) "admin"
    12) "a_long_long_password"

**保存排序结果**

默认情况下， `SORT`_ 操作只是简单地返回排序结果，如果你希望保存排序结果，可以给 ``STORE`` 选项指定一个 ``key`` 作为参数，排序结果将以列表的形式被保存到这个 ``key`` 上。(若指定 ``key`` 已存在，则覆盖。)

::

    redis> EXISTS user_info_sorted_by_level  # 确保指定key不存在
    (integer) 0

    redis> SORT user_id BY user_level_* GET # GET user_name_* GET user_password_* STORE user_info_sorted_by_level    # 排序
    (integer) 12  # 显示有12条结果被保存了

    redis> LRANGE user_info_sorted_by_level 0 11  # 查看排序结果
    1) "59230"
    2) "jack"
    3) "jack201022"
    4) "2"
    5) "huangz"
    6) "nobodyknows"
    7) "222"
    8) "hacker"
    9) "hey,im in"
    10) "1"
    11) "admin"
    12) "a_long_long_password"

一个有趣的用法是将 `SORT`_ 结果保存，用 `EXPIRE`_ 为结果集设置生存时间，这样结果集就成了 `SORT`_ 操作的一个缓存。

这样就不必频繁地调用 `SORT`_ 操作了，只有当结果集过期时，才需要再调用一次 `SORT`_ 操作。

有时候为了正确实现这一用法，你可能需要加锁以避免多个客户端同时进行缓存重建(也就是多个客户端，同一时间进行 `SORT`_ 操作，并保存为结果集)，具体参见 :ref:`setnx` 命令。

**在GET和BY中使用哈希表**

可以使用哈希表特有的语法，在 `SORT`_ 命令中进行 ``GET`` 和 ``BY`` 操作。

::

    # 假设现在我们的用户表新增了一个 serial 项来为作为每个用户的序列号
    # 序列号以哈希表的形式保存在 serial 哈希域内。

    redis> HMSET serial 1 23131283 2 23810573 222 502342349 59230 2435829758
    OK

    # 我们希望以比较 serial 中的大小来作为排序 user_id 的方式

    redis> SORT user_id BY *->serial
    1) "222"
    2) "59230"
    3) "2"
    4) "1"

符号 ``"->"`` 用于分割哈希表的关键字(key name)和索引域(hash field)，格式为 ``"key->field"`` 。

除此之外，哈希表的 ``BY`` 和 ``GET`` 操作和上面介绍的其他数据结构(列表、集合、有序集合)没有什么不同。

**可用版本：**
    >= 1.0.0

**时间复杂度：**
    | O(N+M*log(M))， ``N`` 为要排序的列表或集合内的元素数量， ``M`` 为要返回的元素数量。
    | 如果只是使用 `SORT`_ 命令的 ``GET`` 选项获取数据而没有进行排序，时间复杂度 O(N)。
                               
**返回值：**
    | 没有使用 ``STORE`` 参数，返回列表形式的排序结果。
    | 使用 ``STORE`` 参数，返回排序结果的元素数量。
