- [高可用设计](#高可用设计)
  - [无状态服务高可用设计](#无状态服务高可用设计)
  - [数据库高可用架构设计：主从复制+读写分离](#数据库高可用架构设计主从复制读写分离)
    - [基于数据层的数据库高可用架构](#基于数据层的数据库高可用架构)
    - [基于业务层的数据库高可用架构](#基于业务层的数据库高可用架构)
    - [融合的高可用架构设计](#融合的高可用架构设计)
- [金融级高可用架构](#金融级高可用架构)
  - [复制类型](#复制类型)
- [高可用套件](#高可用套件)
    - [MHA](#mha)
    - [Orchestrator](#orchestrator)
    - [自研数据库管理平台](#自研数据库管理平台)

# 高可用设计

高可用（High Availability）是**系统所能提供无故障服务的一种能力**。 简单地说就是避免因服务器宕机而造成的服务不可用。

系统要达到高可用，一定要做好软硬件的冗余，消除单点故障（SPOF single point of failure）。

冗余是高可用的基础，通常认为，系统投入硬件资源越多，冗余也就越多，系统可用性也就越高。

除了做好冗余，系统还要做好故障转移（Failover）的处理。也就是在最短的时间内发现故障，然后把业务切换到冗余的资源上。

在明确上述高可用设计的基本概念后之后，我们来看一下高可用架构设计的类型：**无状态服务高可用设计、数据库高可用架构设计。**

1. 高可用是系统所能提供无故障服务的一种能力，度量单位是几个 9；
2. 线上系统高可用目标应不低于 99.995%，否则系统频繁宕机，用户体验不好；
3. 高可用实现基础是：冗余 + 故障转移；
4. 无状态服务的高可用设计较为简单，直接故障转移或剔除就行；
5. 数据库作为有状态的服务，设计比较复杂（冗余通过复制技术实现，故障转移需要对应的高可用套件）；
6. 数据库高可用有三大架构设计，请务必牢记这几种设计。

## 无状态服务高可用设计

无状态服务是指在处理请求时不依赖于服务实例的先前状态。这意味着每个请求都可以独立地处理，而不需要依赖于服务实例的内部状态信息。无状态服务通常能够更容易地实现水平扩展，因为请求可以被分发到任何可用的服务实例上，而无需考虑特定请求应该由哪个实例处理。这种特性使得无状态服务更容易实现负载均衡和故障恢复。常见的无状态服务包括 RESTful API 服务和无状态的微服务。

无状态的服务（如 Nginx ）高可用设计非常简单，发现问题直接转移就行，甚至可以通过负载均衡服务，当发现有问题，直接剔除：

![](https://secure2.wostatic.cn/static/tU7msB8BTyoZbXFeLpryMC/image.png?auth_key=1705912710-jQL2zGrqx19wwVHF5GXye7-0-0001fbd3de91776648f830dd260be109)

## 数据库高可用架构设计：主从复制+读写分离

从架构角度看，数据库高可用本身也是业务高可用，所以我们要从业务全流程的角度出发，思考数据库的高可用设计。

我在这里提供了三种数据库的高可用架构设计方法，它们不但适用于 MySQL 数据库，也适用于其他数据库。

### 基于数据层的数据库高可用架构

redis的哨兵模式能实现故障转移

如果MySQL的master服务器挂了，可以通过以下方式进行处理：

1. 启动从服务器（slave）：如果有配置从服务器，可以将从服务器提升为新的主服务器，以便继续提供服务。
2. 修复主服务器：如果主服务器出现故障，可以尝试修复主服务器并将其重新加入集群。
3. 数据库故障转移：使用数据库管理工具进行故障转移，将主服务器的功能切换到备用服务器，以确保服务的持续性。

MySQL并不内建支持哨兵模式。然而，可以使用类似于Percona的工具（Percona XtraDB Cluster）或者Galera Cluster来实现MySQL的高可用性和自动故障转移。这些工具提供了监控和自动化故障转移功能，类似于传统的哨兵模式。

可以发现，我们原先的 Slave3 从服务器提升为了新主机，然后建立了新的复制拓扑架构，Slave2、Slave3 都连到新 Master 进行数据同步。

为了在故障转移后对 Service 服务无感知，所以需要引入 VIP（Virtual IP）虚拟 IP 技术，当发生宕机时，VIP 也需要漂移到新的主服务器。

那么这个架构的真正难点在于：

1. 如何保障数据一致性;  
2. 如何发现主服务器宕机；
3. 故障转移逻辑的处理；

我们可以通过 MySQL 提供的无损复制技术，来保障“数据一致性”。而“发现主服务器宕机”“处理故障转移逻辑”要由数据库高可用套件完成，

![](https://secure2.wostatic.cn/static/7DX3qAcLpGkA48WmtT3ju4/image.png?auth_key=1705912710-bJF9Z83wffsEpesEhvRg38-0-85e9826f68ccfdb8789edd6494ff30f7)

### 基于业务层的数据库高可用架构

当一台数据库主服务器不可用，业务直接写另一台数据库主服务器就可以了。我们来看一下这个架构：

有两个master，数据分别写到两个master，一个故障了，全部写到另一个master中

![](https://secure2.wostatic.cn/static/xgD9RnziR896jCpxUqNUMu/image.png?auth_key=1705912710-kZJ4FQhrpWPhbxRYrohsS9-0-cbbf43d40ddbd5d7778085dfda497c13)

这看似是一种非常简单、粗暴的高可用架构实现方式，但能符合这样设计的业务却并不多，因为该设计前提是**状态可修改**。

比如电商中的订单服务，其基本逻辑就是存储电商业务中每笔订单信息，核心逻辑就是往表Orders 中插入数据，即：

```Python
INSERT INTO Orders(o_orderkey, ... ) VALUES (...)

```

这里 o_orderkey 是主键。为了实现基于业务层的数据库高可用，可以在主键生成过程中加入额外信息，比如服务器编号，这样订单的主键设计变为了：

**PK = 有序UUID-服务器编号**

这样的话，当写入服务器编号 1 时失败了，业务层会把订单的主键修改为服务器编号 2，这样就实现了业务层的高可用，电商中的这种订单号生成方式也称为“跳单”。

而当查询订单信息时，由于主键中包含了服务器编号，那么业务知道该笔订单存储在哪台服务器，就可以非常快速地路由到指定的服务器。

但这样设计的前提是整个服务的写入主键是可以进行跳单设计，且查询全部依赖主键进行搜索。

### 融合的高可用架构设计

将不同编号的订单根据不同的数据库进行存放，比如服务器编号为 1 的订单存放在数据库 DB1 中，服务器编号为 2 的订单存放在数据库 DB2 中。

此外，这里也用到了 MySQL 复制中的部分复制技术，即左上角的主服务器仅将 DB1 中的数据同步到右上角的服务器。同理，右上角的主服务器仅将 DB2 中的数据同步到左上角的服务器。下面的两台从服务器不变，依然从原来的 MySQL 实例中同步数据。

两边写个各自的，然后复制对方的，这样两边都是完整的

![](https://secure2.wostatic.cn/static/uFA5NFeoN1EAMRMXzeRVcR/image.png?auth_key=1705912710-womKaKraT2aow2e31u9wKq-0-c28038906bc2c83061dbed50721f7d95)

# 金融级高可用架构

就是当服务器因为各种原因，发生宕机，导致MySQL 数据库不可用之后，快速恢复业务。但对有状态的数据库服务来说，在一些核心业务系统中，比如电商、金融等，还要保证数据一致性。

这里的“数据一致性”是指在任何灾难场景下，**一条数据都不允许丢失**（一般也把这种数据复制方式叫作“强同步”）。

1. 核心业务复制务必设置为增强半同步复制；
2. 同城容灾使用三园区架构，一地三中心，或者两地三中心，机房见网络延迟不超过 5ms；
3. 跨城容灾使用“三地五中心”，跨城机房距离超过 200KM，延迟超过 25ms；
4. 跨城容灾架构由于网络耗时高，因此一般仅用于读多写少的业务，例如用户中心；
5. 除了复制进行数据同步外，还需要额外的核对程序进行逻辑核对；
6. 数据库层的逻辑核对，可以使用 last_modify_date 字段，取出最近修改的记录。

## 复制类型

开启增强半同步复制

**为了保证主库上的每一个BINLOG事务都能够被可靠地复制到从库**上，主库在每次事务成功提交时，并不及时反馈给前端应用用户，而是**等待至少一个从库也接收到BINLOG事务并成功写入中继日志后，主库才返回Commit**操作成功给客户端

![](https://secure2.wostatic.cn/static/2G7JPiPmmQ8nzjb81ZAsWT/image.png?auth_key=1705912710-iCqEhd1C6nw43Hc4jC9Ln2-0-25f99db6b73f077401a71928180fc0df)

即使启用半同步复制，依然存在当发生主机宕机时，最后一组事务没有上传到从机的可能。图中宕机的主机已经事务到 101，但是从机只接收到事务 100。如果这个时候 Failover，从机提升为主机，那么这时：

![](https://secure2.wostatic.cn/static/dRQC8oZRQCgWSh7h2Ajwqd/image.png?auth_key=1705912710-smCUQZHCuqovkULmigvzae-0-6ee7b8d94929deca0fe95572763e43cf)

可以看到当主从切换完成后，新的 MySQL 开始写入新的事务102，如果这时老的主服务器从宕机中恢复，则这时事务 101 不会同步到新主服务器，导致主从数据不一致。

**但设置 AFTER_SYNC 半同步的好处是，虽然事务 101 在原主机已经有但是未提交，但是在从机没有收到并返回 ACK 前，这个事务对用户是不可见的，所以，用户感受不到事务**。

所以，在做高可用设计时，当老主机恢复时，需要做一次额外的处理，把事务101给“回滚”

# 高可用套件

但是当数据库发生宕机时，MySQL 的主从复制并不会自动地切换，这需要高可用套件对数据库主从进行管理。

MySQL 的高可用套件用于负责数据库的 Failover 操作，也就是当数据库发生宕机时，MySQL 可以剔除原有主机，选出新的主机，然后对外提供服务，保证业务的连续性。

可以看到，MySQL 复制是高可用的技术基础，用于将数据实时同步到从机。高可用套件是MySQL 高可用实现的解决方案，负责切换新主机。

- 为了实现数据切换的透明性，可以采用 VIP 和名字服务机制。VIP 仅用于同机房同网段，名字服务器，比如域名可以跨机房进行切换。
- MySQL 常用的高可用套件有 MHA 和 Orchestrator，它们都能完成 failover 的工作。但是由于管理节点与 MySQL 通信采用 ssh 协议，所以安全性不高，性能也很一般，一般建议用在不超过 20 台数据库节点的环境中。
- 对于要管理 MySQL 数量比较多的场景，推荐自研数据库平台，这样能结合每家公司的不同特性，设计出 MySQL 数据库的自动管理平台，这样才能解放 DBA 的生产力，投入业务的优化工作中去。

VIP（Virtual IP）技术。其中，VIP 不是真实的物理 IP，而是可以随意绑定在任何一台服务器上。

业务访问数据库，不是服务器上与网卡绑定的物理 IP，而是这台服务器上的 VIP。当数据库服务器发生宕机时，高可用套件会把 VIP 插拔到新的服务器上。数据库 Failover后，业务依旧访问的还是 VIP，所以使用 VIP 可以做到对业务透明。

MySQL 的主服务器的 IP 地址是 192.168.1.10，两个从服务器的 IP 地址分别为 192.168.1.20、192.168.1.30。

上层服务访问数据库并没有直接通过物理 IP 192.168.1.10，而是访问 VIP，地址为192.168.1.100。

![](https://secure2.wostatic.cn/static/rrH7gKJ1NJUwhZ7zmSxzqn/image.png?auth_key=1705912710-sXLWnEjS5nYqTwmxJPWQ18-0-549e9a6de4f70fe750d6a463cda7e59a)

这时，如果 MySQL 数据库主服务器发生宕机。当发生 Failover 后，由于上层服务访问的是 VIP 192.168.1.100，所以切换对服务来说是透明的，只是在切换过程中，服务会收到连接数据库失败的提示。但是通过重试机制，当下层数据库完成切换后，服务就可以继续使用了。所以，上层服务一定要做好错误重试的逻辑，否则就算启用 VIP，也无法实现透明的切换

![](https://secure2.wostatic.cn/static/3veF3rMdZW9VS2Lj3KjBq2/image.png?auth_key=1705912710-3ZP3ZphfMhKDHjBRcbeHNz-0-7d42e108a7d7a3adebb996ae123c261d)

 VIP 也是有局限性的，仅限于同机房同网段的 IP 设定。如果是我们之前设计的三园区同城跨机房容灾架构，VIP 就不可用了。这时就要用名字服务，常见的名字服务就是 DNS（Domain Name Service），如下所示：当发生 Failover 后，高可用套件会把域名指向为新的 MySQL 主服务器，IP 地址为202.177.54.20，这样也实现了对于上层服务的透明性。

![](https://secure2.wostatic.cn/static/4cgVUigdzgtToD7snUCoKa/image.png?auth_key=1705912710-jLL3DnAUqJ4d5c6bX7Eucm-0-a606a45c7c7a7a9beb02dc3ded3f4bdc)

### MHA

MHA(Master High Availability)是一款开源的 MySQL 高可用程序，它为 MySQL 数据库主从复制架构提供了 automating master failover 的功能。

MHA Manager 通常部署在一台服务器上（一台服务器部署多个mysql服务），用来判断多个 MySQL 高可用组是否可用。当发现有主服务器发生宕机，就发起 failover 操作。MHA Manger 可以看作是 failover 的总控服务器。

而 MHA Node 部署在每台 MySQL 服务器上，MHA Manager 通过执行 Node 节点的脚本完成failover 切换操作。

MHA Manager 和 MHA Node 的通信是采用 ssh 的方式，也就是需要在生产环境中打通 MHA Manager 到所有 MySQL 节点的 ssh 策略，那么这里就存在潜在的安全风险。

另外，ssh 通信，效率也不是特别高。所以，MHA 比较适合用于规模不是特别大的公司，所有MySQL 数据库的服务器数量不超过 20 台。

![](https://secure2.wostatic.cn/static/dgWKerz6ZStEuhfGbRV9af/image.png?auth_key=1705912710-dqxEw21eqwCkJrQ15ujaTm-0-17e5a8a78883d4e536b617e0385d56d7)

### Orchestrator

Orchestrator 是另一款开源的 MySQL 高可用套件，除了支持 failover 的切换，还可通过Orchestrator 完成 MySQL 数据库的一些简单的复制管理操作。Orchestrator 的开源地址为：[https://github.com/openark/orchestrator](https://github.com/openark/orchestrator)

其基本实现原理与 MHA 是一样的，只是把元数据信息存储在了元数据库中，并且提供了HTTP 接口和命令的访问方式，使用上更为友好。

![](https://secure2.wostatic.cn/static/n2hiCYdx1fs5tjVBZo5bdQ/image.png?auth_key=1705912710-89zzkJEA14qSkPXUwLYRMJ-0-e5ff2daa945b95eee18731a6e4ec01f2)

### 自研数据库管理平台

当然了，虽然 MHA 和 Orchestrator 都可以完成 MySQL 高可用的 failover 操作，但是，在生产环境中如果需要管理成千乃至上万的数据库服务器，由于它们的通信仅采用 ssh 的方式，并不能满足生产上的安全性和性能的要求。

所以，几乎每家互联网公司都会自研一个数据库的管理平台，用于管理公司所有的数据库集群，以及数据库的容灾切换工作。

元数据库用于存储管理 MySQL 数据库所有的节点信息，比如 IP 地址、端口、域名等。

![](https://secure2.wostatic.cn/static/8L3sxkF1coR1LbVeZMJEXr/image.png?auth_key=1705912710-tppv1H1xLFttU6kKzs6EMS-0-3a4f13f4017ab9635cc016444bd0d747)

