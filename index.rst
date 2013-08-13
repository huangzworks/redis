.. Redis命令参考简体中文版 documentation master file, created by
   sphinx-quickstart on Tue Oct 25 17:56:34 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis 命令参考
=================

本文档是 `Redis Command Reference <http://redis.io/commands>`_ 的简体中文翻译版，
原文十一个部分、共一百多个命令已经全部翻译完毕。

所有示例代码均经过 Redis 2.8 版本测试，质量保证。
（\ `查看 Redis 2.8 版本更新内容 <https://redis.readthedocs.org/en/latest/change_log.html#redis-2-8>`_\ ）


命令目录(使用 CTRL + F 快速查找)：
--------------------------------------

+-------------------+-----------------------+-------------------+-----------------------+
| .. toctree::      | .. toctree::          | .. toctree::      | .. toctree::          |
|   :maxdepth: 2    |    :maxdepth: 2       |    :maxdepth: 2   |    :maxdepth: 2       |
|                   |                       |                   |                       |
|   key/index       |    string/index       |    hash/index     |    list/index         |
+-------------------+-----------------------+-------------------+-----------------------+
| .. toctree::      | .. toctree::          | .. toctree::      | .. toctree::          |
|   :maxdepth: 2    |    :maxdepth: 2       |    :maxdepth: 2   |    :maxdepth: 2       |
|                   |                       |                   |                       |
|   set/index       |    sorted_set/index   |    pub_sub/index  |    transaction/index  |
+-------------------+-----------------------+-------------------+-----------------------+
| .. toctree::      | .. toctree::          | .. toctree::                              |
|   :maxdepth: 2    |    :maxdepth: 2       |    :maxdepth: 2                           |
|                   |                       |                                           |
|   script/index    |    connection/index   |    server/index                           |
+-------------------+-----------------------+-------------------+-----------------------+


文档
-------------------

以下文章翻译自 `redis.io/documentation <http://redis.io/documentation>`_ 文档。

+-----------------------+-----------------------+-----------------------+
| .. toctree::          | .. toctree::          | .. toctree::          |
|    :maxdepth: 2       |    :maxdepth: 2       |    :maxdepth: 2       |
|                       |                       |                       |
|    topic/notification |    topic/transaction  |    topic/pubsub       |
+-----------------------+-----------------------+-----------------------+
| .. toctree::          | .. toctree::          | .. toctree::          |
|    :maxdepth: 2       |    :maxdepth: 2       |    :maxdepth: 2       |
|                       |                       |                       |
|    topic/replication  |    topic/protocol     |    topic/persistence  |
+-----------------------+-----------------------+-----------------------+


下载离线版本
------------------

下载： `HTML 格式 <http://media.readthedocs.org/htmlzip/redis/latest/redis.zip>`_

注意，因为文档总是在不断地更新和修正当中，请定期下载最新的离线文档，确保文档的有效性。


关于
-------

本文档由 `huangz <http://huangz.me>`_ 翻译，版权归 Redis 官方所有。

关注 `文档的 github 项目 <https://github.com/huangz1990/redis>`_ 可以随时追踪文档的最新更新，
:doc:`change_log` 记录了文档各个版本的主要更新信息。

有任何问题、意见或建议，可以在文档配套的 disqus 论坛里留言，或者直接联系译者。
