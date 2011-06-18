===
SET
===

SADD
====

``SADD key member``

将member元素加入到集合key当中。

| 如果member元素已经是该集合的成员，那SADD命令不执行任何操作。
| 假如key不存在，则创建一个只包含member元素作成员的集合。
| 当key不是集合类型时，返回一个错误。

时间复杂度:
    O(1)

返回值:
    | 如果添加元素成功，返回1。
    | 如果元素已经是集合的成员，返回0。

::

    redis> SADD bbs "v2ex.com"
    (integer) 1
    redis> SADD bbs "codecompo.com"
    (integer) 1

    redis> SMEMBERS bbs     # 显示bbs集合中所有成员
    1) "codecompo.com"
    2) "v2ex.com"

    redis> SADD bbs "v2ex.com"  # 尝试添加重复元素，返回0
    (integer) 0


SINTER
======

``SINTER key [key ...]``

返回一个集合的全部成员，该集合是所有给定集合的\ **交集**\。

| 不存在的key被视为空集。
| 当给定集合当中有一个空集时，结果也为空集（根据集合运算定律）。

时间复杂度:
    O(N * M)，N为给定集合当中基数最小的集合，M为给定集合的个数。

返回值:
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


SMOVE
=====

``SMOVE source destination member``

将member元素从source集合移动到destination集合。

| SMOVE是原子性操作。
| 如果source集合不存在或不包含指定的member元素，则SMOVE命令不执行任何操作，仅返回0。否则，member元素从source集合中被移除，并添加到destination集合中去。
| 当destination集合已经包含member元素时，SMOVE命令只是简单地将source集合中的member元素删除。
| 当source或destination不是集合类型时，返回一个错误。

复杂度:
    O(1)

返回值:
    | 如果member元素被成功移除，返回1。
    | 如果member元素不是source集合的成员，并且没有任何操作对destination集合执行，那么返回0。

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


SUNION
======

``SUNION key [key ...]``

返回一个集合的全部成员，该集合是所有给定集合的\ **并集**\。

不存在的key被视为空集。

复杂度:
    O(N)，N是所有给定集合的成员数量之和。

返回值:
    并集成员的列表。

::

    redis> SMEMBERS songs
    1) "Billie Jean"

    redis> SMEMBERS my_songs
    1) "Believe Me"

    redis> SUNION songs my_songs
    1) "Billie Jean"
    2) "Believe Me"


SCARD
=====

``SCARD key``

返回集合的\ **基数**\（集合中元素的数量）。

复杂度:
    O(1)

返回值：
    | 集合的基数。
    | 当key不存在时，返回0。

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


SINTERSTORE
===========

``SINTERSTORE destination key [key ...]``

此命令等同于\ `SINTER`_\，但它将结果保存到destination集合，而不是简单地返回结果集。

如果destination集合已经存在，则将其覆盖。

时间复杂度:
    O(N * M)，N为给定集合当中基数最小的集合，M为给定集合的个数。

返回值:
    结果集中的元素数量。

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


SPOP
====

``SPOP key``

移除并返回集合中的一个随机元素。

复杂度:
    O(1)

返回值:
    | 被移除的随机元素。
    | 当key不存在或key是空集时，返回nil。

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


SUNIONSTORE
===========

``SUNIONSTORE destination key [key ...]``


此命令等同于\ `SUNION`_\，但它将结果保存到destination集合，而不是简单地返回结果集。

如果destination已经存在，则将其覆盖。

复杂度:
    O(N)，N是所有给定集合的成员数量之和。

返回值:
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


SDIFF
=====

``SDIFF key [key ...]``

返回一个集合的全部成员，该集合是第一个给定集合和其他所有给定集合的\ **差集** \。

不存在的key被视为空集。

复杂度:
    O(N)，N是所有给定集合的成员数量之和。

返回值:
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


SISMEMBER
=========

``SISMEMBER key member``

判断member元素是否是集合的成员。

时间复杂度:
    O(1)

返回值:
    | 如果member元素是集合的成员，返回1。
    | 如果member元素不是集合的成员，或key不存在，返回0。

::

    redis> SMEMBERS joe's_movies
    1) "hi, lady"
    2) "Fast Five"
    3) "2012"

    redis> SISMEMBER joe's_movies "bet man"
    (integer) 0

    redis> SISMEMBER joe's_movies "Fast Five"
    (integer) 1


SRANDMEMBER
===========

``SRANDMEMBER key``

返回集合中的一个随机元素。

该操作和\ `SPOP`_\相似，但\ `SPOP`_\将随机元素从集合中移除并返回，而\ `SRANDMEMBER`_\则仅仅返回随机元素，而不对集合进行任何改动。

时间复杂度:
    O(1)

返回值:
    被选中的随机元素。
    当key不存在或key是空集时，返回nil。

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


SDIFFSTORE
==========

``SDIFFSTORE destination key [key ...]``

此命令等同于\ `SDIFF`_\，但它将结果保存到destination集合，而不是简单地返回结果集。

如果destination集合已经存在，则将其覆盖。

复杂度:
    O(N)，N是所有给定集合的成员数量之和。

返回值:
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


SMEMBERS
========

``SMEMBERS key``

返回集合中的所有成员。

时间复杂度:
    O(N)，N为集合的基数。

返回值:
    集合中的所有成员。

::

    redis> SMEMBERS prog_lang
    1) "c"
    2) "ruby"
    3) "python"


SREM
====

``SREM key member``

移除集合中的member元素。

| 如果member元素不是集合中的成员，则SREM命令不执行任何操作。
| 当key不是集合类型，返回一个错误。

时间复杂度:
    O(1)

返回值:
    | 如果移除元素成功，返回1。
    | 如果member元素不是集合成员，返回0。

::

    redis> SMEMBERS prog_lang
    1) "c"
    2) "ruby"
    3) "python"

    redis> SREM prog_lang "c"
    (integer) 1

    redis> SMEMBERS prog_lang
    1) "ruby"
    2) "python"

    redis> SREM prog_lang "scheme"
    (integer) 0

    redis> SMEMBERS prog_lang
    1) "ruby"
    2) "python"
