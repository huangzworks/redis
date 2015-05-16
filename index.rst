.. Redis命令参考简体中文版 documentation master file, created by
   sphinx-quickstart on Tue Oct 25 17:56:34 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis 命令参考
=================

.. include:: intro.include

.. include:: tdir.include

命令目录(使用 CTRL + F 快速查找)：
--------------------------------------

+-----------------------+-----------------------+-----------------------+-----------------------+
| .. toctree::          | .. toctree::          | .. toctree::          | .. toctree::          |
|   :maxdepth: 2        |    :maxdepth: 2       |    :maxdepth: 2       |    :maxdepth: 2       |
|                       |                       |                       |                       |
|   key/index           |    string/index       |    hash/index         |    list/index         |
+-----------------------+-----------------------+-----------------------+-----------------------+
| .. toctree::          | .. toctree::          | .. toctree::          | .. toctree::          |
|   :maxdepth: 2        |    :maxdepth: 2       |    :maxdepth: 2       |    :maxdepth: 2       |
|                       |                       |                       |                       |
|   set/index           |    sorted_set/index   |    hyperloglog/index  |    pub_sub/index      |
+-----------------------+-----------------------+-----------------------+-----------------------+
| .. toctree::          | .. toctree::          | .. toctree::          | .. toctree::          |
|    :maxdepth: 2       |   :maxdepth: 2        |    :maxdepth: 2       |    :maxdepth: 2       |
|                       |                       |                       |                       |
|    transaction/index  |   script/index        |    connection/index   |    server/index       |
+-----------------------+-----------------------+-----------------------+-----------------------+

文档
-------------------

以下文章翻译自 `redis.io/documentation <http://redis.io/documentation>`_ 文档。

+-----------------------+---------------------------+-----------------------+
| .. toctree::          | .. toctree::              | .. toctree::          |
|    :maxdepth: 2       |    :maxdepth: 2           |    :maxdepth: 2       |
|                       |                           |                       |
|    topic/notification |    topic/transaction      |    topic/pubsub       |
+-----------------------+---------------------------+-----------------------+
| .. toctree::          | .. toctree::              | .. toctree::          |
|    :maxdepth: 2       |    :maxdepth: 2           |    :maxdepth: 2       |
|                       |                           |                       |
|    topic/replication  |    topic/protocol         |    topic/persistence  |
+-----------------------+---------------------------+-----------------------+
| .. toctree::          | .. toctree::              | .. toctree::          |
|    :maxdepth: 2       |    :maxdepth: 2           |    :maxdepth: 2       |
|                       |                           |                       |
|    topic/sentinel     |    topic/cluster-tutorial |    topic/cluster-spec |
+-----------------------+---------------------------+-----------------------+


.. include:: about.include

.. include:: donation.include
