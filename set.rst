.. _set_struct:

集合(Set)
**********

附录，常用集合运算：

::

    A = {'a', 'b', 'c'}
    B = {'a', 'e', 'i', 'o', 'u'}

    inter(x, y): 交集，在集合x和集合y中都存在的元素。
    inter(A, B) = {'a'}
    
    union(x, y): 并集，在集合x中或集合y中的元素，如果一个元素在x和y中都出现，那只记录一次即可。
    union(A,B) = {'a', 'b', 'c', 'e', 'i', 'o', 'u'}

    diff(x, y): 差集，在集合x中而不在集合y中的元素。
    diff(A,B) = {'b', 'c'}

    card(x): 基数，一个集合中元素的数量。
    card(A) = 3

    空集： 基数为0的集合。


.. _sadd:

SADD
=====

.. function:: SADD key member [member ...]

将一个或多个\ ``member``\ 元素加入到集合\ ``key``\ 当中，已经存在于集合的\ ``member``\ 元素将被忽略。

假如\ ``key``\ 不存在，则创建一个只包含\ ``member``\ 元素作成员的集合。

当\ ``key``\ 不是集合类型时，返回一个错误。

**时间复杂度:**
    O(N)，\ ``N``\ 是被添加的元素的数量。

**返回值:**
    被添加到集合中的\ *新*\ 元素的数量，不包括被忽略的元素。

.. note:: 在Redis2.4版本以前，\ `SADD`_\ 只接受单个\ ``member``\ 值。

::

    # 添加单个元素

    redis> SADD bbs "discuz.net"
    (integer) 1

    # 添加重复元素 

    redis> SADD bbs "discuz.net"
    (integer) 0

    # 添加多个元素

    redis> SADD bbs "tianya.cn" "groups.google.com"
    (integer) 2

    redis> SMEMBERS bbs
    1) "discuz.net"
    2) "groups.google.com"
    3) "tianya.cn"


.. _srem:

SREM
=====

.. function:: SREM key member [member ...]

移除集合\ ``key``\ 中的一个或多个\ ``member``\ 元素，不存在的\ ``member``\ 元素会被忽略。

当\ ``key``\ 不是集合类型，返回一个错误。

**时间复杂度:**
    O(N)，\ ``N``\ 为给定\ ``member``\ 元素的数量。

**返回值:**
    被成功移除的元素的数量，不包括被忽略的元素。

.. note:: 在Redis2.4版本以前，\ `SREM`_\ 只接受单个\ ``member``\ 值。

::

    # 测试数据

    redis> SMEMBERS languages
    1) "c"
    2) "lisp"
    3) "python"
    4) "ruby"

    # 移除单个元素

    redis> SREM languages ruby
    (integer) 1

    # 移除不存在元素

    redis> SREM languages non-exists-language
    (integer) 0

    # 移除多个元素

    redis> SREM languages lisp python c
    (integer) 3

    redis> SMEMBERS languages
    (empty list or set)


.. _smembers:

SMEMBERS
=========

.. function:: SMEMBERS key

返回集合\ ``key``\ 中的所有成员。

**时间复杂度:**
    O(N)，\ ``N``\ 为集合的基数。

**返回值:**
    集合中的所有成员。

::

    # 情况1：空集合

    redis> EXISTS not_exists_key    # 不存在的key视为空集合
    (integer) 0

    redis> SMEMBERS not_exists_key
    (empty list or set)

    
    # 情况2：非空集合

    redis> SADD programming_language python
    (integer) 1

    redis> SADD programming_language ruby
    (integer) 1

    redis> SADD programming_language c
    (integer) 1

    redis> SMEMBERS programming_language
    1) "c"
    2) "ruby"
    3) "python"


.. _sismember:

SISMEMBER
==========

.. function:: SISMEMBER key member

判断\ ``member``\ 元素是否是集合\ ``key``\ 的成员。

**时间复杂度:**
    O(1)

**返回值:**
    | 如果\ ``member``\ 元素是集合的成员，返回\ ``1``\ 。
    | 如果\ ``member``\ 元素不是集合的成员，或\ ``key``\ 不存在，返回\ ``0``\ 。

::

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"

    redis> SISMEMBER joe's_movies "bet man"
    (integer) 0

    redis> SISMEMBER joe's_movies "Fast Five"
    (integer) 1


.. _scard:

SCARD
======

.. function:: SCARD key

返回集合\ ``key``\ 的\ **基数**\(集合中元素的数量)。

**时间复杂度:**
    O(1)

**返回值：**
    | 集合的基数。
    | 当\ ``key``\ 不存在时，返回\ ``0``\ 。

::

    redis> SMEMBERS tool
    1) "pc"
    2) "printer"
    3) "phone"

    redis> SCARD tool
    (integer) 3

    redis> SMEMBERS fake_set
    (empty list or set)

    redis> SCARD fake_set
    (integer) 0


.. _smove:

SMOVE
========

.. function:: SMOVE source destination member

将\ ``member``\ 元素从\ ``source``\ 集合移动到\ ``destination``\ 集合。

\ `SMOVE`_\ 是原子性操作。

如果\ ``source``\ 集合不存在或不包含指定的\ ``member``\ 元素，则\ `SMOVE`_\ 命令不执行任何操作，仅返回\ ``0``\ 。否则，\ ``member``\ 元素从\ ``source``\ 集合中被移除，并添加到\ ``destination``\ 集合中去。

当\ ``destination``\ 集合已经包含\ ``member``\ 元素时，\ `SMOVE`_\ 命令只是简单地将\ ``source``\ 集合中的\ ``member``\ 元素删除。

当\ ``source``\ 或\ ``destination``\ 不是集合类型时，返回一个错误。

**时间复杂度:**
    O(1)

**返回值:**
    | 如果\ ``member``\ 元素被成功移除，返回\ ``1``\ 。
    | 如果\ ``member``\ 元素不是\ ``source``\ 集合的成员，并且没有任何操作对\ ``destination``\ 集合执行，那么返回\ ``0``\ 。

::

    redis> SMEMBERS songs
    1) "Billie Jean"
    2) "Believe Me"

    redis> SMEMBERS my_songs
    (empty list or set)

    redis> SMOVE songs my_songs "Believe Me"
    (integer) 1

    redis> SMEMBERS songs
    1) "Billie Jean"

    redis> SMEMBERS my_songs
    1) "Believe Me"


.. _spop:

SPOP
=====

.. function:: SPOP key

移除并返回集合中的一个随机元素。

**时间复杂度:**
    O(1)

**返回值:**
    | 被移除的随机元素。
    | 当\ ``key``\ 不存在或\ ``key``\ 是空集时，返回\ ``nil``\ 。

::

    redis> SMEMBERS my_sites
    1) "huangz.iteye.com"
    2) "sideeffect.me"
    3) "douban.com/people/i_m_huangz"

    redis> SPOP my_sites
    "huangz.iteye.com"  

    redis> SMEMBERS my_sites
    1) "sideeffect.me"
    2) "douban.com/people/i_m_huang"

.. seealso:: 如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用\ `SRANDMEMBER`_\ 命令。


.. _srandmember:

SRANDMEMBER
============

.. function:: SRANDMEMBER key

返回集合中的一个随机元素。

该操作和\ `SPOP`_\相似，但\ `SPOP`_\将随机元素从集合中移除并返回，而\ `SRANDMEMBER`_\则仅仅返回随机元素，而不对集合进行任何改动。

**时间复杂度:**
    O(1)

**返回值:**
    被选中的随机元素。
    当\ ``key``\ 不存在或\ ``key``\ 是空集时，返回\ ``nil``\ 。

::

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"

    redis> SRANDMEMBER joe's_movies
    "Fast Five"

    redis> SMEMBERS joe's_movies    # 集合中的元素不变
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"


.. _sinter:

SINTER
========

.. function:: SINTER key [key ...]

返回一个集合的全部成员，该集合是所有给定集合的\ *交集*\。

不存在的\ ``key``\ 被视为空集。

当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。

**时间复杂度:**
    O(N * M)，\ ``N``\ 为给定集合当中基数最小的集合，\ ``M``\ 为给定集合的个数。

**返回值:**
    交集成员的列表。

::

    redis> SMEMBERS group_1
    1) "LI LEI"
    2) "TOM"
    3) "JACK"   # <-

    redis> SMEMBERS group_2
    1) "HAN MEIMEI"
    2) "JACK"   # <- 

    redis> SINTER group_1 group_2
    1) "JACK"


.. _sinterstore:

SINTERSTORE
============

.. function:: SINTERSTORE destination key [key ...]

此命令等同于\ `SINTER`_\，但它将结果保存到\ ``destination``\ 集合，而不是简单地返回结果集。

如果\ ``destination``\ 集合已经存在，则将其覆盖。

\ ``destination``\ 可以是\ ``key``\ 本身。

**时间复杂度:**
    O(N * M)，\ ``N``\ 为给定集合当中基数最小的集合，\ ``M``\ 为给定集合的个数。

**返回值:**
    结果集中的成员数量。

::

    redis> SMEMBERS songs
    1) "good bye joe"   # <-
    2) "hello,peter"

    redis> SMEMBERS my_songs
    1) "good bye joe"   # <-
    2) "falling"

    redis> SINTERSTORE song_and_my_song songs my_songs
    (integer) 1

    redis> SMEMBERS song_and_my_song
    1) "good bye joe"


.. _sunion:

SUNION
=======

.. function:: SUNION key [key ...]

返回一个集合的全部成员，该集合是所有给定集合的\ *并集*\。

不存在的\ ``key``\ 被视为空集。

**时间复杂度:**
    O(N)，\ ``N``\ 是所有给定集合的成员数量之和。

**返回值:**
    并集成员的列表。

::

    redis> SMEMBERS songs
    1) "Billie Jean"

    redis> SMEMBERS my_songs
    1) "Believe Me"

    redis> SUNION songs my_songs
    1) "Billie Jean"
    2) "Believe Me"


.. _sunionstore:

SUNIONSTORE
============

.. function:: SUNIONSTORE destination key [key ...]


此命令等同于\ `SUNION`_\，但它将结果保存到\ ``destination``\ 集合，而不是简单地返回结果集。

如果\ ``destination``\ 已经存在，则将其覆盖。

\ ``destination``\ 可以是\ ``key``\ 本身。

**时间复杂度:**
    O(N)，\ ``N``\ 是所有给定集合的成员数量之和。

**返回值:**
    结果集中的元素数量。

::

    redis> SMEMBERS ms_sites
    1) "microsoft.com"
    2) "skype.com"

    redis> SMEMBERS google_sites
    1) "youtube.com"
    2) "google.com"

    redis> SUNIONSTORE google_and_ms_sites ms_sites google_sites
    (integer) 4

    redis> SMEMBERS google_and_ms_sites
    1) "microsoft.com"
    2) "skype.com"
    3) "google.com"
    4) "youtube.com"


.. _sdiff:

SDIFF
======

.. function:: SDIFF key [key ...]

返回一个集合的全部成员，该集合是所有给定集合的\ *差集* \。

不存在的\ ``key``\ 被视为空集。

**时间复杂度:**
    O(N)，\ ``N``\ 是所有给定集合的成员数量之和。

**返回值:**
    交集成员的列表。

::

    redis> SMEMBERS peter's_movies
    1) "bet man"
    2) "start war"
    3) "2012"   # <-

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"   # <-

    redis> SDIFF peter's_movies joe's_movies
    1) "bet man"
    2) "start war"


.. _sdiffstore:

SDIFFSTORE
============

.. function:: SDIFFSTORE destination key [key ...]

此命令等同于\ `SDIFF`_\，但它将结果保存到\ ``destination``\ 集合，而不是简单地返回结果集。

如果\ ``destination``\ 集合已经存在，则将其覆盖。

\ ``destination``\ 可以是\ ``key``\ 本身。

**时间复杂度:**
    O(N)，\ ``N``\ 是所有给定集合的成员数量之和。

**返回值:**
    结果集中的元素数量。

::

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"

    redis> SMEMBERS peter's_movies
    1) "bet man"
    2) "start war"
    3) "2012"

    redis> SDIFFSTORE joe_diff_peter joe's_movies peter's_movies
    (integer) 2

    redis> SMEMBERS joe_diff_peter
    1) "hi, lady"
    2) "Fast Five"
