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

.. include:: set/sadd.rst

.. include:: set/srem.rst

.. include:: set/smembers.rst

.. include:: set/sismember.rst

.. include:: set/scard.rst

.. include:: set/smove.rst

.. include:: set/spop.rst

.. include:: set/srandmember.rst

.. include:: set/sinter.rst

.. include:: set/sinterstore.rst

.. include:: set/sunion.rst

.. include:: set/sunionstore.rst

.. include:: set/sdiff.rst

.. include:: set/sdiffstore.rst


