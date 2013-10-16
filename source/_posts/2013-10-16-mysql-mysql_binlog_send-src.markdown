---
layout: post
title: "Mysql rpl_master.cc:mysql_binlog_send 源码的一些个人分析和吐槽"
date: 2013-10-16 22:50
comments: true
categories:  mysql replication source_code
---
读了两天rpl_master.cc:mysql_binlog_send的源码（Mysql 5.6.11），总结一下

函数的入口是rpl_master.cc:com_binlog_dump，当slave向master向master请求数据时，在master上调用

函数参数说明。log_ident为slave请求的binlog文件名，如"mysql-bin.000001"。pos为slave请求的binlog位置。slave_gtid_executed为gtid相关，在此忽略

在此吐槽：

1. 这个函数将近1k行，且缩进混乱，代码折叠困难。最后附的我的笔记中，有整理好的源码下载
2. 这个函数有两大段近百行的重复代码（1179 & 1553）

#源码的主体结构

{% codeblock 源码的主体结构 lang:java %}
mysql_binlog_send(…)
{
     0814 … bla bla
     1011 fake_rotate_event
     1028 max_alloed_packet= MAX_MAX_ALLOWED_PACKET
     1038 if (请求的POS不是从binlog开头开始)
     1039 {
               从binlog开头中找到一个FD event(FORMAT_DESCRIPTION_EVENT), 并发送给slave
     1123 }
     1124 else
     1125 {
               FD event可以从正常的replication中传送给slave，此处不做操作
     1127 }
     1132 while (net和the都在运转)
     1133 {
     1143      while (从binlog中读取一个event)
     1144      {
     1178           switch (event_type)
     1179           {
                         分类型处理event
     1281           }
     1283           若event需跳转到下一个binlog(goto_next_binlog), break
     1291           fire HOOK before_send_event
     1300           记录skip_group
     1306           {
                         send last skip group heartbeat?
     1326           }
     1331           向slave发送event
     1348           {
                         处理LOAD_EVENT
     1356           }
     1358           fire HOOK after_send_event
     1369      }
     1391      if (!goto_next_binlog)
     1392      {
                   发送完所有binlog，未发生binlog切换时
     1437          加锁尝试再读取一个event（此时其他进程不能更新binlog），目的是试探之前处理过程中master上是否有更多的binlog写入，若有，则跳转1553处理read_packet
     1451          若没有更多的binlog 
                   {
                        等待更多的binlog写入，等待时发送心跳
     1545          }
     1553          处理read_packet
                   {                         
                        分类型处理event
     1682          }
     1683      }
     1685      if (goto_next_binlog)
               {
                    切换到下一个binlog
               }
     1733 }
     1735 之后是收尾处理
}
{% endcodeblock %}

#重点步骤
1. 关于Format Description event，如果传送从binlog开头开始，那么FD event会正常随着binlog传送；若传送不从binlog开头开始，则需要补发一个FD event，才开始传送
2. 如何判断binlog读取完。函数先不加锁读取binlog中的event，读完后，再加锁尝试读取一个event（加锁过程中，没有其他进程写进binlog），若有数据，则继续处理，若没有数据，则说明binlog读取完了，master会阻塞等待新的binlog写入。这样做主要为了：1. 不需要一直加锁读取binlog，保障性能；2. 无锁读取时会有其他进程写binlog，加锁可以保障这些新加的binlog得到妥善安置
3. 心跳。心跳尽在不传送binlog时（master穷尽了binlog，开始阻塞等待新的binlog写入时）才进行心跳
4. Fake Rotate Event。Fake Rotate Event在开始传送和切换binlog时发送到slave。主要作用是通知slave binlog filename，原因在源码comment里写的很清楚。但是很疑惑的是为什么在FD event里并没有binlog filename，这个问题发到了[StackoverFlow](http://stackoverflow.com/questions/19375951/in-mysql-replication-why-format-description-event-doesnt-include-binlogs-name)，未有答案。（诶，看看我的stackoverflow的记录就知道，我的问题都是死题）

#TODO
有一些东西还是没弄懂，得慢慢读懂其他机制才可以，比如

1. max_alloed_packet是如何作用的
2. send last skip group heartbeat的作用
3. 不同类型的event的具体处理，需要和slave端结合在一起

#我的笔记
我的笔记[在此](https://app.yinxiang.com/shard/s11/sh/f23e9619-9c3d-47f5-a911-8945d0ee02a5/f4eb8539fb2f99e1481496c994b2c270)