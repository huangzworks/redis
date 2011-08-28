.. _key_strcut:

键(Key)
********

.. _del:

DEL
====

.. function:: DEL key [key ...]

移除给定的一个或多个\ ``key``\ 。

如果\ ``key``\ 不存在，则忽略该命令。

**时间复杂度：**
    | O(N)，\ ``N``\ 为要移除的\ ``key``\ 的数量。

    | 移除单个字符串类型的\ ``key``\ ，时间复杂度为O(1)。
    | 移除单个列表、集合、有序集合或哈希表类型的\ ``key``\ ，时间复杂度为O(M)，\ ``M``\ 为以上数据结构内的元素数量。

**返回值：**
    被移除\ ``key``\ 的数量。

::

    # 情况1： 删除单个key

    redis> SET name huangz # 设置一个值
    OK

    redis> DEL name  # 将其删除
    (integer) 1


    # 情况2： 删除一个不存在的key

    redis> EXISTS phone  # 检查一个不存在的key
    (integer) 0

    redis> DEL phone # 试图删除一个不存在的key，失败
    (integer) 0


    # 情况3： 同时删除多个key

    redis> MSET name huangz age 20 blog huangz.iteye.com # 同时设置多个key-value对
    OK

    redis> DEL name age blog # 同时删除三个key
    (integer) 3


.. _keys:

KEYS
=====

.. function:: KEYS pattern

查找符合给定模式的\ ``key``\ 。

| \ ``KEYS *``\ 命中数据库中所有\ ``key``\ 。
| \ ``KEYS h?llo``\ 命中\ ``hello``\ ， \ ``hallo and hxllo``\ 等。
| \ ``KEYS h*llo``\ 命中\ ``hllo``\ 和\ ``heeeeello``\ 等。
| \ ``KEYS h[ae]llo``\ 命中\ ``hello``\ 和\ ``hallo``\ ，但不命中\ ``hillo``\ 。

特殊符号用\ ``"\"``\ 隔开

**时间复杂度：**
    O(N)，\ ``N``\ 为数据库中\ ``key``\ 的数量。
            
**返回值：**
    符合给定模式的\ ``key``\ 列表。

.. warning::
    \ `KEYS`_\ 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如果你需要从一个数据集中查找特定的\ ``key``\ ，你最好还是用\ :ref:`set_struct`\ 。

::

    redis> mset one 1 two 2 three 3 four 4  # 一次设置4个key
    OK

    redis> keys *o*
    1) "four"
    2) "two"
    3) "one"
    
    redis> keys t??
    1) "two"
    
    redis> keys t[w]*
    1) "two"
    
    redis> keys *  # 匹配数据库内所有key
    1) "four"
    2) "three"
    3) "two"
    4) "one"


.. _randomkey:

RANDOMKEY
==========

**RANDOMKEY**

从当前数据库中随机返回(不删除)一个\ ``key``\ 。

**时间复杂度：**
    O(1)

**返回值：**
    | 当数据库不为空时，返回一个\ ``key``\ 。
    | 当数据库为空时，返回\ ``nil``\ 。

:: 

    # 情况1：数据库不为空

    redis> mset fruit "apple" drink "beer" food "cookies"   # 设置多个key
    OK

    redis> randomkey
    "fruit"

    redis> randomkey
    "food"

    redis> keys *    # 查看数据库内所有key，证明RANDOMKEY并不删除key
    1) "food"
    2) "drink"
    3) "fruit"


    # 情况2：数据库为空

    redis> flushdb  # 删除当前数据库所有key
    OK

    redis> randomkey
    (nil)


.. _ttl:

TTL
====

.. function:: TTL key

返回给定\ ``key``\ 的剩余生存时间(time to live)(以秒为单位)。

**时间复杂度：**
    O(1)

**返回值：**
    | \ ``key``\ 的剩余生存时间(以秒为单位)。
    | 当\ ``key``\ 不存在或没有设置生存时间时，返回\ ``-1``\  。

::

    # 情况1：带TTL的key

    redis> set name "huangz"    # 设置一个key
    OK

    redis> expire name 30  # 设置生存时间为30秒
    (integer) 1

    redis> get name
    "huangz"

    redis> ttl name
    (integer) 25

    redis> ttl name # 30秒过去，name过期
    (integer) -1

    redis> get name # 过期的key将被删除
    (nil)


    # 情况2：不带TTL的key

    redis> SET site wikipedia.org   
    OK

    redis> TTL wikipedia.org
    (integer) -1


    # 情况3：不存在的key

    redis> EXISTS not_exists_key
    (integer) 0

    redis> TTL not_exists_key
    (integer) -1


.. _exists:

EXISTS
======

.. function:: EXISTS key

检查给定\ ``key``\ 是否存在。

**时间复杂度：**
    O(1)

**返回值：**
    若\ ``key``\ 存在，返回\ ``1``\ ，否则返回\ ``0``\ 。

::

    redis> set db "redis"
    OK

    redis> exists db  # key存在
    (integer) 1

    redis> del db   # 删除key
    (integer) 1

    redis> exists db  # key不存在
    (integer) 0


.. _move:

MOVE
====

.. function:: MOVE key db

将当前数据库(默认为\ ``0``\ )的\ ``key``\ 移动到给定的数据库\ ``db``\ 当中。

如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定\ ``key``\ ，或者\ ``key``\ 不存在于当前数据库，那么\ ``MOVE``\ 没有任何效果。

因此，也可以利用这一特性，将\ `MOVE`_\ 当作锁(locking)原语。

**时间复杂度：**
    O(1)

**返回值：**
    移动成功返回\ ``1``\ ，失败则返回\ ``0``\ 。

::

    # 情况1： key存在于当前数据库

    redis> SELECT 0  # redis默认使用数据库0，为了清晰起见，这里再显式指定一次。
    OK

    redis> SET song "secret base - Zone"
    OK

    redis> MOVE song 1  # 将song移动到数据库1
    (integer) 1

    redis> EXISTS song  # song已经被移走
    (integer) 0

    redis> SELECT 1  # 使用数据库1
    OK

    redis:1> EXISTS song  # 证实song被移到了数据库1(注意命令提示符变成了"redis:1"，表明正在使用数据库1)
    (integer) 1
 

    # 情况2：当key不存在的时候 

    redis:1> EXISTS fake_key  
    (integer) 0

    redis:1> MOVE fake_key 0  # 试图从数据库1移动一个不存在的key到数据库0，失败
    (integer) 0

    redis:1> select 0  # 使用数据库0
    OK

    redis> EXISTS fake_key  # 证实fake_key不存在
    (integer) 0


    # 情况3：当源数据库和目标数据库有相同的key时

    redis> SELECT 0  # 使用数据库0
    OK

    redis> SET favorite_fruit "banana"
    OK

    redis> SELECT 1  # 使用数据库1
    OK
    redis:1> SET favorite_fruit "apple"
    OK

    redis:1> SELECT 0  # 使用数据库0，并试图将favorite_fruit移动到数据库1
    OK

    redis> MOVE favorite_fruit 1  # 因为两个数据库有相同的key，MOVE失败
    (integer) 0
    
    redis> GET favorite_fruit  # 数据库0的favorite_fruit没变
    "banana"

    redis> SELECT 1
    OK

    redis:1> GET favorite_fruit  # 数据库1的favorite_fruit也是
    "apple"


.. _rename:

RENAME
=======

.. function:: RENAME key newkey

将\ ``key``\ 改名为\ ``newkey``\ 。

当\ ``key``\ 和\ ``newkey``\ 相同或者\ ``key``\ 不存在时，返回一个错误。

当\ ``newkey``\ 已经存在时，\ `RENAME`_\ 命令将覆盖旧值。

**时间复杂度：**
    O(1)

**返回值：**
    改名成功时提示\ ``OK``\ ，失败时候返回一个错误。

:: 

    # 情况1：key存在且newkey不存在

    redis> SET message "hello world"
    OK
    
    redis> RENAME message greeting
    OK

    redis> EXISTS message  # message不复存在
    (integer) 0
    
    redis> EXISTS greeting  # greeting取而代之
    (integer) 1


    # 情况2：当key不存在时，返回错误
    
    redis> RENAME fake_key never_exists
    (error) ERR no such key
    

    # 情况3：newkey已存在时，RENAME会覆盖旧newkey
    
    redis> SET pc "lenovo"
    OK
    
    redis> SET personal_computer "dell"
    OK

    redis> RENAME pc personal_computer
    OK

    redis> GET pc
    (nil)

    redis:1> GET personal_computer  # dell“没有”了
    "lenovo"


.. _type:

TYPE
=====

.. function:: TYPE key

返回\ ``key``\ 所储存的值的类型。

**时间复杂度：**
    O(1)

**返回值：**
    | \ ``none``\ (key不存在)
    | \ ``string``\ (字符串)
    | \ ``list``\ (列表)
    | \ ``set``\ (集合)
    | \ ``zset``\ (有序集)
    | \ ``hash``\ (哈希表)

::

    redis> SET weather "sunny"  # 构建一个字符串
    OK

    redis> TYPE weather 
    string

    redis> LPUSH book_list "programming in scala"  # 构建一个列表
    (integer) 1

    redis> TYPE book_list 
    list

    redis> SADD pat "dog"  # 构建一个集合
    (integer) 1

    redis> TYPE pat
    set


.. _expire:

EXPIRE
=======

.. function:: EXPIRE key seconds

为给定\ ``key``\ 设置生存时间。

当\ ``key``\ 过期时，它会被自动删除。

在Redis中，带有生存时间的\ ``key``\ 被称作“易失的”(volatile)。

| 在低于2.1.3版本的Redis中，已存在的生存时间不可覆盖。
| 从2.1.3版本开始，\ ``key``\ 的生存时间可以被更新，也可以被\ `PERSIST`_\ 命令移除。(详情参见 http://redis.io/topics/expire)。

**时间复杂度：**
    O(1)

**返回值：**
    | 设置成功返回\ ``1``\ 。
    | 当\ ``key``\ 不存在或者不能为\ ``key``\ 设置生存时间时(比如在低于2.1.3中你尝试更新\ ``key``\ 的生存时间)，返回\ ``0``\ 。

::

    redis> SET cache_page "www.twitter.com/huangz1990"
    OK
    
    redis> EXPIRE cache_page 30  # 设置30秒后过期
    (integer) 1
    
    redis> TTL cache_page   # 查看给定key的剩余生存时间
    (integer) 24
    
    redis> EXPIRE cache_page 30000  # 更新生存时间，30000秒
    (integer) 1
    
    redis> TTL cache_page
    (integer) 29996
    
   
.. _object:

OBJECT
======

.. function:: OBJECT subcommand [arguments [arguments]]

\ `OBJECT`_\ 命令允许从内部察看给定\ ``key``\ 的Redis对象。

| 它通常用在除错(debugging)或者了解为了节省空间而对\ ``key``\ 使用特殊编码的情况。
| 当将Redis用作缓存程序时，你也可以通过\ `OBJECT`_\ 命令中的信息，决定\ ``key``\ 的驱逐策略(eviction policies)。

OBJECT命令有多个子命令：

* \ ``OBJECT REFCOUNT <key>``\ 返回给定\ ``key``\ 引用所储存的值的次数。此命令主要用于除错。
* \ ``OBJECT ENCODING <key>``\ 返回给定\ ``key``\ 锁储存的值所使用的内部表示(representation)。
* \ ``OBJECT IDLETIME <key>``\ 返回给定\ ``key``\ 自储存以来的空转时间(idle， 没有被读取也没有被写入)，以秒为单位。

| 对象可以以多种方式编码：

* 字符串可以被编码为\ ``raw``\ (一般字符串)或\ ``int``\ (用字符串表示64位数字是为了节约空间)。
* 列表可以被编码为\ ``ziplist``\ 或\ ``linkedlist``\ 。\ ``ziplist``\ 是为节约大小较小的列表空间而作的特殊表示。
* 集合可以被编码为\ ``intset``\ 或者\ ``hashtable``\ 。\ ``intset``\ 是只储存数字的小集合的特殊表示。
* 哈希表可以编码为\ ``zipmap``\ 或者\ ``hashtable``\ 。\ ``zipmap``\ 是小哈希表的特殊表示。
* 有序集合可以被编码为\ ``ziplist``\ 或者\ ``skiplist``\ 格式。\ ``ziplist``\ 用于表示小的有序集合，而\ ``skiplist``\ 则用于表示任何大小的有序集合。

| 假如你做了什么让Redis没办法再使用节省空间的编码时(比如将一个只有1个元素的集合扩展为一个有100万个元素的集合)，特殊编码类型(specially encoded types)会自动转换成通用类型(general type)。
                                                                                                        
**时间复杂度：**
    O(1)

**返回值：**
    | \ ``REFCOUNT``\ 和\ ``IDLETIME``\ 返回数字。
    | \ ``ENCODING``\ 返回相应的编码类型。

::

    redis> SET game "COD"  # 设置一个字符串
    OK
    
    redis> OBJECT REFCOUNT game  # 只有一个引用
    (integer) 1
    
    redis> OBJECT IDLETIME game  # 等待一阵。。。然后查看空转时间
    (integer) 90
    
    redis> GET game  # 提取game， 让它处于活跃(active)状态
    "COD"

    redis> OBJECT IDLETIME game  # 不再处于空转
    (integer) 0

    redis> OBJECT ENCODING game  # 字符串的编码方式
    "raw"

    redis> SET phone 15820123123  # 大的数字也被编码为字符串
    OK

    redis> OBJECT ENCODING phone
    "raw"

    redis> SET age 20  # 短数字被编码为int
    OK
    
    redis> OBJECT ENCODING age
    "int"


.. _renamenx:

RENAMENX
=========

.. function:: RENAMENX key newkey

当且仅当\ ``newkey``\ 不存在时，将\ ``key``\ 改为\ ``newkey``\ 。

出错的情况和\ `RENAME`_\ 一样(\ ``key``\ 不存在时报错)。

**时间复杂度：**
    O(1)

**返回值：**
    | 修改成功时，返回\ ``1``\ 。
    | 如果\ ``newkey``\ 已经存在，返回\ ``0``\ 。

::

    # 情况1：newkey不存在，成功

    redis> SET player "MPlyaer"
    OK

    redis> EXISTS best_player
    (integer) 0

    redis> RENAMENX player best_player
    (integer) 1


    # 情况2：newkey存在时，失败

    redis> SET animal "bear"
    OK

    redis> SET favorite_animal "butterfly"
    OK

    redis> RENAMENX animal favorite_animal
    (integer) 0

    redis> get animal
    "bear"

    redis> get favorite_animal
    "butterfly"


.. _expireat:

EXPIREAT
========

.. function:: EXPIREAT key timestamp

\ `EXPIREAT`_\ 的作用和\ `EXPIRE`_\ 一样，都用于为\ ``key``\ 设置生存时间。

不同在于\ `EXPIREAT`_\ 命令接受的时间参数是\ *UNIX时间戳*\ (unix timestamp)。

**时间复杂度：**
    O(1)

**返回值：**
    | 如果生存时间设置成功，返回\ ``1``\ 。
    | 当\ ``key``\ 不存在或没办法设置生存时间，返回\ ``0``\ 。

::

    redis> SET cache www.google.com
    OK

    redis> EXPIREAT cache 1355292000 # 这个key将在2012.12.12过期
    (integer) 1

    redis> TTL cache
    (integer) 45081860


.. _persist:

PERSIST
========

.. function:: PERSIST key

移除给定\ ``key``\ 的生存时间。

**时间复杂度：**
    O(1)

**返回值：**
    | 当生存时间移除成功时，返回\ ``1``\ .
    | 如果\ ``key``\ 不存在或\ ``key``\ 没有设置生存时间，返回\ ``0``\ 。

::

    redis> SET time_to_say_goodbye "886..."
    OK

    redis> EXPIRE time_to_say_goodbye 300
    (integer) 1

    redis> TTL time_to_say_goodbye
    (integer) 293
    
    redis> PERSIST time_to_say_goodbye  # 移除生存时间
    (integer) 1
    
    redis> TTL time_to_say_goodbye  # 移除成功
    (integer) -1


.. _sort:

SORT
=====

.. function:: SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination]

返回或保存给定列表、集合、有序集合\ ``key``\ 中经过排序的元素。

排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。

**一般SORT用法**

最简单的\ `SORT`_\ 使用方法是\ ``SORT key``\ 。

假设\ ``today_cost``\ 是一个保存数字的列表，\ `SORT`_\ 命令默认会返回该列表值的递增(从小到大)排序结果。

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

当数据集中保存的是字符串值时，你可以用\ ``ALPHA``\ 修饰符(modifier)进行排序。
   
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

如果你正确设置了\ ``!LC_COLLATE``\ 环境变量的话，Redis能识别\ ``UTF-8``\ 编码。

| 排序之后返回的元素数量可以通过\ ``LIMIT``\ 修饰符进行限制。
| \ ``LIMIT``\ 修饰符接受两个参数：\ ``offset``\ 和\ ``count``\ 。
| \ ``offset``\ 指定要跳过的元素数量，\ ``count``\ 指定跳过\ ``offset``\ 个指定的元素之后，要返回多少个对象。

以下例子返回排序结果的前5个对象(\ ``offset``\ 为\ ``0``\ 表示没有元素被跳过)。

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

修饰符可以组合使用。以下例子返回降序(从大到小)的前5个对象。

:: 

    redis> SORT rank LIMIT 0 5 DESC
    1) "78"
    2) "67"
    3) "56"
    4) "55"
    5) "50"

**使用外部key进行排序**

有时候你会希望使用外部的\ ``key``\ 作为权重来比较元素，代替默认的对比方法。

假设现在有用户(user)数据如下：

    =====  ====== ======
    id     name   level
    =====  ====== ======
    1      admin   9999
    2      huangz  10   
    59230  jack    3   
    222    hacker  9999 
    =====  ====== ======

| \ ``id``\ 数据保存在\ ``key``\ 名为\ ``user_id``\ 的列表中。
| \ ``name``\ 数据保存在\ ``key``\ 名为\ ``user_name_{id}``\ 的列表中
| \ ``level``\ 数据保存在\ ``user_level_{id}``\ 的\ ``key``\ 中。

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

如果希望按\ ``level``\ 从大到小排序\ ``user_id``\ ，可以使用以下命令：

::

    redis> SORT user_id BY user_level_* DESC
    1) "222"    # hacker
    2) "1"      # admin
    3) "2"      # huangz    
    4) "59230"  # jack

但是有时候只是返回相应的\ ``id``\ 没有什么用，你可能更希望排序后返回\ ``id``\ 对应的用户名，这样更友好一点，使用\ ``GET``\ 选项可以做到这一点：

::

    redis> SORT user_id BY user_level_* DESC GET user_name_*
    1) "hacker"
    2) "admin"
    3) "huangz"
    4) "jack"

可以多次地、有序地使用\ ``GET``\ 操作来获取更多外部\ ``key``\ 。

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

\ ``GET``\ 还有一个特殊的规则——\ ``"GET #"``\ ，用于获取被排序对象(我们这里的例子是\ ``user_id``\ )的当前元素。

比如你希望\ ``user_id``\ 按\ ``level``\ 排序，还要列出\ ``id``\ 、\ ``name``\ 和\ ``password``\ ，可以使用以下命令：

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
    
\ ``BY``\ 修饰符可以将一个不存在的\ ``key``\ 当作权重，让\ `SORT`_\ 跳过排序操作。

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

默认情况下，\ `SORT`_\ 操作只是简单地返回排序结果，如果你希望保存排序结果，可以给\ ``STORE``\ 选项指定一个\ ``key``\ 作为参数，排序结果将以列表的形式被保存到这个\ ``key``\ 上。(若指定\ ``key``\ 已存在，则覆盖。)

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

一个有趣的用法是将\ `SORT`_\ 结果保存，用\ `EXPIRE`_\ 为结果集设置生存时间，这样结果集就成了\ `SORT`_\ 操作的一个缓存。

这样就不必频繁地调用\ `SORT`_\ 操作了，只有当结果集过期时，才需要再调用一次\ `SORT`_\ 操作。

有时候为了正确实现这一用法，你可能需要加锁以避免多个客户端同时进行缓存重建(也就是多个客户端，同一时间进行\ `SORT`_\ 操作，并保存为结果集)，具体参见\ :ref:`setnx`\ 命令。

**在GET和BY中使用哈希表**

可以使用哈希表特有的语法，在\ `SORT`_\ 命令中进行\ ``GET``\ 和\ ``BY``\ 操作。

::

    # 假设现在我们的用户表新增了一个serial项来为作为每个用户的序列号
    # 序列号以哈希表的形式保存在serial哈希域内。

    redis> HMSET serial 1 23131283 2 23810573 222 502342349 59230 2435829758
    OK

    # 我们希望以比较serial中的大小来作为排序user_id的方式

    redis> SORT user_id BY *->serial
    1) "222"
    2) "59230"
    3) "2"
    4) "1"

符号\ ``"->"``\ 用于分割哈希表的关键字(key name)和索引域(hash field)，格式为\ ``"key->field"``\ 。

除此之外，哈希表的\ ``BY``\ 和\ ``GET``\ 操作和上面介绍的其他数据结构(列表、集合、有序集合)没有什么不同。

**时间复杂度：**
    | O(N+M*log(M))，\ ``N``\ 为要排序的列表或集合内的元素数量，\ ``M``\ 为要返回的元素数量。
    | 如果只是使用\ `SORT`_\ 命令的\ ``GET``\ 选项获取数据而没有进行排序，时间复杂度O(N)。
                               
**返回值：**
    | 没有使用\ ``STORE``\ 参数，返回列表形式的排序结果。
    | 使用\ ``STORE``\ 参数，返回排序结果的元素数量。
