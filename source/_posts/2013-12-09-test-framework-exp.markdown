---
layout: post
title: "关于DEV测试的一些经验总结"
date: 2013-12-09 20:05
comments: true
categories:  test
---

已经是第三遍重写项目主代码了，这几天用主代码和测试框架互相补充。总结下经验值，以期升级

---
关于测试框架，重写了几遍仍然保留下来的功重要能是：

0. 插桩代码，后面的功能也都依赖插桩。不在主代码中插桩，测试基本靠拜神；

1. 条件池，根据条件池，代码才能沿着需要的分支进行；

2. 签到点（checkpoint），代码要能根据测试要求停得下来，等待状态，之后跑得起来；

3. 外缘测试。比起直接测试变量，还是分析log比较容易维护

测试的难点是：

0. 资源回收。要连续跑测试，资源回收是说说容易的事。tcp server关了，listener停了，关闭的那些connection会不会瞬时占用临时端口；如果某一个connection正在申请二步锁，测试停止时远端锁失败，近端锁如果不释放会不会影响之后的测试；如果系统会自动重启tcp server，tcp server是在测试关闭操作之前停还是之后停还是停两次。想想头就大了。

1. 想停都停不下来，万一断言失败，测试要能停下来。整个flow上都要处理停止中断，用“硬”中断很难掌握资源回收的状况；用“模拟”中断，所有等待/超时/重试的地方都要处理，逐级退栈。比起不测试的代码，主代码花在错误处理的代码量要大很多，但是值得。

2. 测试界限把握不易。测粗了没作用，测细了耗时间而且波动大。插桩的深度，模拟中断的层级，这些都要拿好轻重。否则代码已腐败。

每次跑测试，比起旁边坐个QA的测试，更步步惊心。QA一天才跑十几个case，自动的话2-3分钟就跑十几个简化的case，出错的概率要大很多。 

最后反思一下，如果靠人肉测，...