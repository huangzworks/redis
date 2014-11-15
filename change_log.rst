.. _change_log:

更新日志(change log)
=========================

2013 年 12 月(Redis 2.8)
-----------------------------

Redis 2.8 带来的更新：

- 添加 `SCAN` 命令、 `SSCAN` 命令、 `HSCAN` 命令和 `ZSCAN` 命令。

- 添加 `config_rewrite` 命令。

- 添加 `pubsub` 命令。

- 添加 `client_setname` 命令和 `client_getname` 命令。 

- 修改 `ttl` 和 `pttl` 命令的返回值。

文档本身的更新：

- 开始翻译 Redis 的主题（topic）文档：
  目前已经完成了 `cluster_tutorial` 、 `sentinel` 、 `replication_topic` 、 `persistence` 等主要文档，
  未来还会翻译更多主题文档。

- 支持 PDF 格式的离线文档下载。

- 添加捐款连接。

2012 年 4 月(Redis 2.6)
--------------------------

此次更新的主要内容来自于 Redis 2.6 版本：

- 2.6 版本新增的所有命令(EVAL 、 SCRIPT * 、 TIME 、 PTTL 等)的相关文档全部翻译完毕。
- 2.6 版本新添加的几个应用模式，比如 INCR 命令的计数器模式和限速器模式、 EXPIRE 命令的导航会话模型等，全部翻译完毕。
- 2.6 版本对旧有命令的改进，比如为 SHUTDOWN 添加修饰符、为 INFO 设置新的输出格式 、 CONFIG RESETSTAT 的变动等，这些命令的文档全部翻译/更新完毕。
- 对代码示例及其注释进行了很多修改和重排版，让代码示例更直观。

文档自身的更新：

- 添加 disqus 评论功能。
- 命令不再按类型分页，而是每个命令各一页，载入速度更快。
- 添加进度表，随时了解项目最新动态。

2011 年 12 月(Redis 2.4)
--------------------------

完成 pub/sub 、 transaction 、 connection 和 server 四个部分的翻译。

2011 年 10 月(Redis 2.4)
--------------------------

更新命令到 Redis 2.4 版本。

2011 年 6 月(Redis 2.2)
--------------------------

完成 keys 、 string 、 list 、 set 和 sorted set 六个部分的翻译。

2011 年 4 月(Redis 2.2)
--------------------------

开始进行 Redis 命令参考的翻译工作。
