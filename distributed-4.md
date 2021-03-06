#分布式服务框架的4项特性

![](images/dis/service-architecture.jpg)


在移动及云时代，尽管大部分可扩展的问题可以通过云平台解决，但是服务本身的扩展性挑战仍然存在。比如一个新的项目，用PHP或JSP实现了基本功能，部署在Apache或Tomcat等容器上，在业界这种部署在一个容器内的功能模块通常可以称为一个service。服务容器很容易通过EC2或者docker等方式来扩展部署更多的实例。但service本身的管理的以下几个方面的问题仍然需要架构师去设计及解决。

1、服务的远程调用（RPC）。

随着系统的用户访问规模增大，以及系统功能的增多，众多的功能模块(service)很难用单个service来承载，这些不同功能的service可能由不同的开发团队开发，甚至使用不同的开发语言，最终部署在不同的服务器容器内。Service之间的调用需要一种协议及远程调用的实现，需要具备灵活的data type支持，对调用双方透明（理想情况它就像在执行本地调用），并且具有良好的性能。比较典型的RPC实现有
Google: Protocol Buffers RPC
Facebook: Thrift
Twitter: Finagle

2、服务的分布式调用链及服务状态跟踪统计

随着系统内部的服务增多，一个功能的调用可能会通过系统内部多个服务相互调用来完成，并且这些服务可能由不同的团队开发，并且分布在不同的服务器容器，甚至可能在多个地域不同的机房内。因此分布式系统需要有一种方式来清晰的了解系统的调用及运行状况，测量系统的运行性能，方便准确的指导系统的优化及改进。

由于trace的主要功能都是依赖日志输出来完成，因此通常也需要建设相关的分布式日志系统及数据实时分析展示系统等，分布式日志收集及数据实时分析也是一个非常大的话题，本文不展开详谈。比较典型的trace系统有
Google: Dapper
Twitter: zipkin

3、服务的配置管理。包括服务发现、负载均衡及服务依赖管理。

比较常用的服务发现是域名方式，调用方通过域名来定位目标服务。域名寻址方式可通过配置多IP（或VIP）来实现负载均衡。域名方式存在一些弊端，常用的DNS服务器通常是上个世纪的产物，管理繁琐，更新域名记录后由于协议设计上存在的TTL缓存机制，修改不能立即生效，也无法做变更的push操作。更高级的特性如控制流量、灰度发布等也无法实现。

目前，成熟的分布式服务较多使用基于ZooKeepr的配置服务，ZooKeeper由于与client保持长连，因此具有push能力，可以迅速的调整配置及生效。但由于ZooKeeper本身只是一个通用工具，分布式服务具体场景各种高级特性还需要自行在此基础上实现。

基于DNS的服务发现在负载均衡方面具有明显缺点，由于多IP或VIP使用类似round robin的策略，在实践中，同一服务的多个IP之间负载通常并不均衡。而服务提供方并没有有效手段对runtime期间不均衡进行调整。基于ZooKeeper的配置服务可以支持各种复杂的特性，但需要做一些定制化开发，但服务发现作为系统中最核心的一个环节，这些定制化开发需要较多的经验的积累及沉淀，之前在线上就碰到过配置服务将一个服务池中疑似不稳定的节点移除过多，从而导致出现雪崩的情况。

除了ZooKeeper之外，业界还有一些类似工具，如serfdom及consui。

4、服务之间的调度及生命周期管理

目前大部分服务的部署都是按照事先的规划安装在机房不同的服务器上，配置服务通常只是起服务节点的failover作用，业务中真正按弹性调度来运作的系统还不普遍。但原理上所有的service可以看做是MapReduce中的task，它的调度及生命周期可以很高效的由分布式容器来管理，并且根据service的属性来灵活的分配资源，比如控制CPU的核及内存大小。

业界比较成熟的有Apache旗下的Mesos及YARN。它们的特性有点类似，但是由不同的组织开发。Mesos主要功能是由C++来实现，可以支持docker container来进行调度，因此它的实现更偏底层一些。Yarn是Hadoop 2的一部分，可以更灵活的来调度MapReduce jobs，它是一种JVM内部的实现，可以很好的支持JVM级别的任务分配及调度。

上面介绍了这么多，主要是最近考虑团队在上述1-4之间做一些事情。一方面目前业界在这几点之间还有一些缺失或者欠优化之处，另外1-4点之间也可以适当做一些实现的整合。整合并不是要产出一个大而庞杂的软件，我个人是极力反对大而全，也不喜欢沉重的框架，业务的service实现方不应该import太多工具或者SDK，因此将要做的功能肯定是透明及可插拔的。

由于这件事情并没有严格的可参考目标，因此对于最终实现的是哪个子集还存在一些不确定因素，这个项目如果具有通用性，也考虑以开源的方式来开发。对这方面有什么想法，欢迎留言。
