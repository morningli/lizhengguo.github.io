# ZooKeeper典型使用场景一览

[原文链接](https://blog.csdn.net/qq_27529917/article/details/80640807)

ZooKeeper是一个高可用的分布式数据管理与系统协调框架。基于对Paxos算法的实现，使该框架保证了分布式环境中数据的强一致性，也正是基于这样的特性，使得zookeeper能够应用于很多场景。网上对zk的使用场景也有不少介绍，本文将结合作者身边的项目例子，系统的对zk的使用场景进行归类介绍。 值得注意的是，zk并不是生来就为这些场景设计，都是后来众多开发者根据框架的特性，摸索出来的典型使用方法。因此，也非常欢迎你分享你在ZK使用上的奇技淫巧。

![](https://img-blog.csdn.net/2018061014023718?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI3NTI5OTE3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)