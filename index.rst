.. Redis Command Reference (Chinese Translation) documentation master file, created by
   sphinx-quickstart on Fri Jun 17 11:45:59 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis命令参考中文版(Redis Command Reference)
=============================================

本文翻译自\ `Redis Command Reference <http://redis.io/commands>`_\ ，是Redis命令参考的中文翻译版。

全文共分为十个部分，其中主要的六个部分(Key、String、Hash、List、Set、SortedSet)的所有命令已经翻译完毕，剩余的四个部分(Pub/Sub、Transactions、Connection、Server)还有待日后跟进。

| 几乎重写了所有代码，其一是因为Redis官方大量使用\ ``mylist``\ 、\ ``mystring``\ 等无意义名字，为清晰起见改写了代码示例；
| 其二是补齐了一些Redis官方没有覆盖到的命令或命令的特殊情况。所有示例代码经过Redis 2.2.10版本测试，质量保证。

欢迎传播本文地址，谢绝并鄙视全文转载等微创新行为。

| 报告翻译错误或加入本翻译项目，请到\ `项目GitHub页面 <https://github.com/huangz1990/redis>`_\ 。
| 联系译者请到\ `twitter <http://twitter.com/#!/huangz1990>`_\ 、\ `豆瓣 <http://www.douban.com/people/i_m_huangz/>`_\ 或发送邮件到Gmail：huangz1990 (友情提示：使用"#"替换"@"对于防范垃圾邮件一点效果也没有)

最后更新自: 2011年7月16日。

**目录(使用CTRL+F快速查找命令)：**

.. toctree::
   :maxdepth: 2
   
   key
   string
   hash
   list
   set
   sorted_set

