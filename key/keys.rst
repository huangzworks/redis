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
