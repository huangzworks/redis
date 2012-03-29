.. _list_struct:

表(List)
*********

**头元素和尾元素**

头元素指的是列表左端/前端第一个元素，尾元素指的是列表右端/后端第一个元素。

举个例子，列表\ ``list``\ 包含三个元素：\ ``x, y, z``\ ，其中\ ``x``\ 是头元素，而\ ``z``\ 则是尾元素。

**空列表**

指不包含任何元素的列表，Redis 将不存在的\ ``key``\ 也视为空列表。

.. include:: list/lpush.rst

.. include:: list/lpushx.rst

.. include:: list/rpush.rst

.. include:: list/rpushx.rst

.. include:: list/lpop.rst

.. include:: list/rpop.rst

.. include:: list/blpop.rst

.. include:: list/brpop.rst

.. include:: list/llen.rst

.. include:: list/lrange.rst

.. include:: list/lrem.rst

.. include:: list/lset.rst

.. include:: list/ltrim.rst

.. include:: list/lindex.rst

.. include:: list/linsert.rst

.. include:: list/rpoplpush.rst

.. include:: list/brpoplpush.rst
