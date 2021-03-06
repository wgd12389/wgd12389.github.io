---
layout: post
title: (转)分布式系统原理
category: 技术
tags: [tech]
keywords: 分布式
description: 本文介绍了分布式系统相关的原理知识
---




## 一、分布式系统基础重要要点：  

1. 对外提供无状态节点，内部实现具体有状态或者无状态节点逻辑，节点即可以是提供服务，也可以是存储数据。  
2. 拜占庭问题，在分布式系统中的使用，目的是保证服务可用，而不是找出错误的节点，如果。  
3. 异常常见情况，机器宕机、网络异常、消息丢失、消息乱序、数据错误、不可靠的TCP。可能是收到消息后宕机、也可能是处理完成以后机器宕机、处理完成任务后发送确认消息是网络异常。也有可能是发出去的消息丢失，或者发送确认消息时丢失。可能先发送出去的数据后收到  
4. 分布式状态、成功、失败、超时。超时的情况，不能判断是否成功，原有同上。  
5. 数据存储在机械硬盘上，随时有可能发生异常，导致数据没有能正确存储。  
6. 无法归类的异常，比如，系统的处理能力时高、时低，的诡异行为。  
7. 即使是小概率事件，在每天百万、千万、及以上的运算量时也会上升为大概率事件。  
8. 副本提高数据的冗余，提高系统的可用性，但是在使用副本代来好处的同时，也导致维护副本需要成本。如副本的一致性，多个副本一致性，多个副本直接可能到不一致。  
9. 一致性级别：强一致性、单调一致性，读取最新数据、会话一致性，通过版本读取统一值。最终一致性、弱一致性。  
 
1. 分布式系统性能指标：吞吐量、响应延迟、并发量；常用单位QPS，即每秒钟的处理能力。高吞吐量会带来低响应、他们之间是相互制约关系。  
2. 可用性指标：可以服务时间和非服务时间的比率和请求的成功和失败次数来衡量。  
3. 可扩展性指标：实现能水平扩展，增加低配置的机器即可以实现更大的运算量，和更高的处理能力。  
4. 一致性指标：实现副本间的一致性能力，一致性需要严格考量是否业务允许。  

## 二、分布式系统原理：  

1. 哈希方式，把不同的值进行哈希运算，映射到，不同的机器或者节点。考虑冗余的时候可以把多个哈希值映射到同一个地方。哈希的实现方式，取余。其实现扩展时，比较困难，数据分散在很多机器上，扩展的时候要从个机器上获取数据。而且容易出现分布不均有的情况。  

	> 常见的哈希，用IP、URL、ID、或者固定的值进行哈希，总是得到相同的结果。  

2. 按数据范围分布，比如ID在1~20的在机器A上，ID在21~40的在机器B上，ID在40~60的在机器C上实现，ID在60~100的分布在机器D 上，数据分布比较均匀。如果某个节点处理能力有限，可以直接分裂该节点。维护数据分布的元信息，可能出现单点瓶颈。几千机器，每个机器又划分为N个范围， 导致需要维护的数据分布范围元数据过大，导致可能需要几台机器实现。   

	> 一定要严格控制元数据量，进可能的减少元数据的存储。    
    
3. 按数据量分布，另一类常用的数据分布方式则是按照数据量分布数据。与哈希方式和按数据范围方式不同，数据量分布数据与具体的数据特征无关，而是将数据视为 一个顺序增长的文件，并将这个文件按照某一较为固定的大小划分为若干数据块（chunk），不同的数据块分布到不同的服务器上。与按数据范围分布数据的方 式类似的是，按数据量分布数据也需要记录数据块的具体分布情况，并将该分布信息作为元数据使用元数据服务器管理。  

	> 由于与具体的数据内容无关，按数据量分布数据的方式一般没有数据倾斜的问题，数据总是被均匀切分并分布到集群中。当集群需要重新负载均衡时，只需通过迁移 数据块即可完成。集群扩容也没有太大的限制，只需将部分数据库迁移到新加入的机器上即可以完成扩容。按数据量划分数据的缺点是需要管理较为复杂的元信息， 与按范围分布数据的方式类似，当集群规模较大时，元信息的数据量也变得很大，高效的管理元信息成为新的课题。   
	
4. 一致性哈希，构造哈希环，有哈希域[0,10]，则构造3个部分，[1,4)/[4,9)/[9,10),[0,1)/分成了3个部分，这3部分是一个环 状，增加机器时，变动的是其附近的节点，分担的是附近节点的压力，其元数据的维护和按数据量分布一样。其未来的扩展，可以实现多个需节点。  
5. 构建映射元数据，建立映射表的方式。  
6. 副本与数据分布，把一个数据副本分散到多台服务器上。比如应用A的数据，存储在A、B、C ，3台机器上，如果3台机器中，其中一台出现问题，请求被处理到其他2台机器上，如果加机器恢复，还需要从另外2台机器上，Copy数据，又增加了这2台 机器的负担。如果我们有应用A和应用B，各自有3台机器，那么我们可以把A应用分散在6台机器上，B应用也分散在6台机器上，可以实现相同的数据备份，但 是应用存储的数据被分散了。某台机器损害，只是把该机器所承担的负载平均分配到了，另外5台机器上。恢复数据从5台机器恢复，其速度快和给各台服务器的压 力都不大，而且可以实现机器损害，更换完全不影响应用。  

	> 其原理是多个机器互为副本，是比较理想的实现负载分压的方式。  
	
7. 分布式计算思想，移动数据不如移动计算，就进计算原则，减少跨进程、跨网络、等跨度较大的实现，把计算所需的资源尽可能的靠近。因为可能出现网络、远程机器的瓶颈。  
8. 常见分布式系统数据分布方式: GFS、HDFS:按数据量分布；Map reduce 按GFS的数据分布做本地化；BigTable、HBase按数据范围分布；Pnuts按哈希方式或者数据范围分布，可以选择；Dynamo、 Cassndra按一致性哈希；Mola、Armor、BigPipe按哈希方式分布；Doris按哈希方式和按数据量分布组合。  

## 三、数据副本协议  

1. 副本一定要满足一定的可用性和一致性要求、具体容错能力，即使出现一些问题也能提供可靠服务。  
2. 数据副本的基本协议，中心化和去中心化2种基本的副本控制协议。  
3. 中心化副本控制协议的基本思路是由一个中心节点协调副本数据的更新、维护副本之间的一致性。中心化副本控制协议的优点是协议相对较为简单，所有的副本相关 的控制交由中心节点完成。并发控制将由中心节点完成，从而使得一个分布式并发控制问题，简化为一个单机并发控制问题。控制问题，简化为一个单机并发控制问 题。所谓并发控制，即多个节点同时需要修改副本数据时，需要解决“写写”、“读写”等并发冲突。单机系统上常用加锁等方式进行并发控制。对于分布式并发控 制，加锁也是一个常用的方法，但如果没有中心节点统一进行锁管理，就需要完全分布式化的锁系统，会使得协议非常复杂。中心化副本控制协议的缺点是系统的可 用性依赖于中心化节点，当中心节点异常或与中心节点通信中断时，系统将失去某些服务（通常至少失去更新服务），所以中心化副本控制协议的缺点正是存在一定 的停服务时间。即存在单点问题，即使中心化节点是一个集群，也只不过是一个大的单点。  
4. 副本数据同步常见问题:   

	> 1)网络异常，导致副本没有得到数据；  
	> 2）数据脏读，主节点数据已经更新，但是由于某种原因，没有得到最新数据    
	> 3）增加新节点没有得到主节点数据，而读数据时从新节点读数据导致，没有得到数据。  
	
5. 去中心化副本控制协议没有中心节点，协议中所有的节点都是完全对等的，节点之间通过平等协商达到一致。从而去中心化协议没有因为中心化节点异常而带来的停 服务等问题。然而，没有什么事情是完美的，去中心化协议的最大的缺点是协议过程通常比较复杂。尤其当去中心化协议需要实现强一致性时，协议流程变得复杂且 不容易理解。由于流程的复杂，去中心化协议的效率和性能较低。   
6. Paxos是唯一在工程中得到应用的强一致性去中心化副本控制协议。ZooKeeper、Chubby，就是该协议的应用。   
	> Zookeeper用Paxos协议选择Leader，用Lease协议控制数据是否有效。用Quorum协议把Leader的数据同步到follow。  
	> Zeekeeper，实现Quorum写入时，如果没有完全写入成功，则所有的follow机器，反向向Leader写数据，写入数据后 follow又向Leader同步数据，保持一致，如果是失败的数据先写入，你们follow同步到原来的数据，相对于回滚；如是是最新的数据先写入 Leader则就是完成了最新数据的更新。  
 
7. Megastore，使用的是改进型行Paxos协议。  
8. Dynamo / Cassandra使用基于一致性哈希的去中心化协议。Dynamo使用Quorum机制来管理副本。  
9. Lease机制是最重要的分布式协议，广泛应用于各种实际的分布式系统中。1）Lease通常定义为：颁发者在一定期限内給予持有者一定权利的协议。 2）Lease 表达了颁发者在一定期限内的承诺，只要未过期颁发者必须严格遵守 lease 约定的承诺；3）Lease 的持有者在期限内使用颁发者的承诺，但 lease 一旦过期必须放弃使用或者重新和颁发者续约。4）的影响。中心服务器发出的lease的含义为：在lease的有效期内，中心服务器保证不会修改对应数据 的值。5）可以通过版本号、过多上时间、或者到某个固定时间点认为Lease证书失效。  

	> 其原理和我们的Cache一样，比如浏览器缓存道理一致。其要求时间时钟同步，因为数据完全依赖于期限。  
	
10. 心跳（heartbeat）检测不可靠,假如检测及其Q，被检测机器A，可能由于Q发起检测，但是A的回应被阻塞，导致Q       认为A宕机，阻塞很快恢复，导致根据心跳检测来做判断不可靠；也可能是他们之间的网络断开；也可能是Q机器本身异常导致认为A机器宕机；如果根据Q的检测 结果，来判断很可能出现多个主机的情况。  
11. Write-all-read-one（简称WARO）是一种最简单的副本控制规则，顾名思义即在更新时写所有的副本，只有在所有的副本上更新成功，才认 为更新成功，从而保证所有的副本一致，这样在读取数据时可以读任一副本上的数据。写多份，读从其中一份读取。  
12. quorum协议，其实就是读取成功的副本数大于失败的副本数，你们读取的副本里面一定包含了最新的副本。  
13. Mola*和Armor*系统中所有的副本管理都是基于Quorum，即数据在多数副本上更新成功则认为成功。  
14. Big Pipe*中的副本管理也是采用了WARO机制。  
 
## 四、日志技术  

1. 日志技术是宕机恢复的主要技术之一。日志技术最初使用在数据库系统中。严格来说日志技术不是一种分布式系统的技术，但在分布式系统的实践中，却广泛使用了日志技术做宕机恢复，甚至如BigTable等系统将日志保存到一个分布式系统中进一步增强了系统容错能力。  
2. 两种比较实用的日志技术Redo Log与No Redo/No undo Log。  
3. 数据库的日志主要分为Undo Log、Redo Log、Redo/Undo Log与No Redo/No Undo Log。这四类日志的区别在更新日志文件和数据文件的时间点要求不同，从而造成性能和效率也不相同。  
4. 本节介绍另一种特殊的日志技术“No Undo/No Redo log”，这种技术也称之为“0/1目录”(0/1 directory)。还有一个主记录，记录当前工作目录，比如老数据在0目录下，新数据在1目录下，我们访问数据时，通过主纪录，记录当前是工作在那个 目录下，如果是工作在目录0下，取目录0数据，反之取1目录数据。  
5. MySQL的主从库设计也是基于日志。从库只需通过回放主库的日志，就可以实现与主库的同步。由于从库同步的速度与主库更新的速度没有强约束，这种方式只能实现最终一致性。  
6. 在单机上，事务靠日志技术或MVCC等技术实现。  
7. 两阶段提交的思路比较简单，在第一阶段，协调者询问所有的参与者是否可以提交事务（请参与者投票），所有参与者向协调者投票。在第二阶段，协调者根据所有 参与者的投票结果做出是否事务可以全局提交的决定，并通知所有的参与者执行该决定。在一个两阶段提交流程中，参与者不能改变自己的投票结果。两阶段提交协 议的可以全局提交的前提是所有的参与者都同意提交事务，只要有一个参与者投票选择放弃(abort)事务，则事务必须被放弃。可以这么认为，两阶段提交协 议对于这种超时的相关异常也没有很好的容错机制，整个流程只能阻塞在这里，且对于参与者而言流程状态处于未知，参与者即不能提交本地节点上的事务，也不能 放弃本地节点事务。  
8. 第一、两阶段提交协议的容错能力较差。  
9. 第二、两阶段提交协议的性能较差。一次成功的两阶段提交协议流程中，协调者与每个参与者之间至少需要两轮交互4个消息“prepare”、“vote- commit”、“global-commit”、“确认global-commit”。过多的交互次数会降低性能。另一方面，协调者需要等待所有的参与 者的投票结果，一旦存在较慢的参与者，会影响全局流程执行速度。  
10. 顾名思义，MVCC即多个不同版本的数据实现并发控制的技术，其基本思想是为每次事务生成一个新版本的数据，在读数据时选择不同版本的数据即可以实现对事 务结果的完整性读取。在使用MVCC时，每个事务都是基于一个已生效的基础版本进行更新，事务可以并行进行。其思想是根据版本号，在多个节点取同一个版本 号的数据。  
11. MVCC的流程过程非常类似于SVN等版本控制系统的流程，或者说SVN等版本控制系统就是使用的MVCC思想。  

## 五、CAP理论   

1. CAP理论的定义很简单，CAP三个字母分别代表了分布式系统中三个相互矛盾的属性：  

	> Consistency (一致性)：CAP理论中的副本一致性特指强一致性（1.3.4 ）  
	> Availiablity(可用性)：指系统在出现异常时已经可以提供服务；  
	> Toleranceto the partition of network (分区容忍)：指系统可以对网络分区，这种异常情况进行容错处理。  
	   
2. CAP理论指出：无法设计一种分布式协议，使得同时完全具备CAP三个属性，即1)该种协议下的副本始终是强一致性，2)服务始终是可用的，3)协议可以容忍任何网络分区异常；分布式系统协议只能在CAP这三者间所有折中。    



> 作者：狼-志  
> 链接：http://www.cnblogs.com/gowhy/archive/2012/12/28/2837399.html  
> 来源：cnblogs  
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。  

