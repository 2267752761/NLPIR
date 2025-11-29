学习计划：https://github.com/2267752761/NLPIR/blob/master/mivlus.md


1、学习：向量基础（欧几里得距离、余弦相似度、内积）✅
欧几里得距离：两个向量之间的直线距离。dist(x,y)=sqrt(∑(xi-yi)^2)
余弦相似度：两个向量之间的夹角余弦值
内积：两个向量对应元素乘积的和。x·y
实践：安装 Docker / K8s 环境 ✅

2、学习：Milvus 架构（Proxy、QueryNode、DataNode 等）✅
第一层：访问层
- 无状态（使用Nginx、Kubernetes Ingress、NodePort 和 LVS 等负载均衡组件提供统一的服务地址）
- 由于 Milvus 采用的是大规模并行处理（MPP）架构，代理会对中间结果进行聚合和后处理，然后再将最终结果返回给客户端。
第二层：协调器-mivlus的大脑
其部分任务如下：
- DDL/DCL/TSO 管理：处理数据定义语言 (DDL) 和数据控制语言 (DCL) 请求，以及管理时间戳 Oracle (TSO) 和时间刻度签发。
- 流服务管理：将先写日志（WAL）与流节点绑定，并为流服务提供服务发现功能。
- 查询管理：管理查询节点的拓扑结构和负载平衡，并提供和管理服务查询视图，以指导查询路由。
- 历史数据管理：将压缩和索引建立等离线任务分配给数据节点，并管理数据段和数据视图的拓扑结构。
第三层：工作节点-协调器的哑执行器，无状态
- 流节点：作为碎片级的"小型大脑"，基于底层WAL存储提供碎片级的一致性保证和故障恢复。同时，流节点还负责增长数据查询和生成查询计划。此外，它还负责将增长数据转换为封存（历史）数据。
- 查询节点：查询节点从对象存储中加载历史数据，并提供历史数据查询。
- 数据节点：数据节点负责离线处理历史数据，如压缩和建立索引。
第四层：存储
- 元存储：元存储存储元数据快照，如 Collections Schema 和消息消耗检查点。元数据的存储要求极高的可用性、强一致性和事务支持，因此 Milvus 选择 etcd 作为元存储。Milvus 还使用 etcd 进行服务注册和健康检查。
- 对象存储：对象存储用于存储日志快照文件、标量和向量数据的索引文件以及中间查询结果。
- WAL存储：先写日志（WAL）存储是分布式系统中数据持久性和一致性的基础。在提交任何更改之前，首先要将其记录在日志中，以确保在发生故障时，可以准确恢复到之前的位置。通过提前记录每次写入操作，WAL 层保证了可靠的全系统恢复和一致性机制--无论你的分布式环境发展得多么复杂。
如何实现计算节点（QueryNode / DataNode 等）的横向扩展？
- Milvus横向扩展计算节点的核心依赖Coord的自动注册和任务分配机制，结合数据分片策略。用户只需启动新节点并连接到集群，Coord会自动感知并负载均衡，Proxy层透明处理客户端请求，实现无缝扩展。
实践：用 Docker 启动 Milvus ✅

3、学习：Collection、Partition、FieldSchema ✅
实践：创建 Collection 并插入向量 ✅

4、CRUD 操作流程
实践：实现 insert / delete / search（Flat 索引）

5、向量维度和存储大小关系 ✅
mivlus中向量维度和存储大小的关系主要取决于向量类型、维度大小以及存储的附加信息（ID、索引、元数据等）。
- 基本存储计算公式
假设有N个向量，每个向量维度为D，向量类型为float32(单精度浮点数，4字节)：
向量存储大小(Bytes) = N * D * 4
- 向量类型对存储的影响，如float32、float64、binary
- 附加存储开销
除了原始向量，还需要考虑ID列（默认int64，每条向量8字节）、索引结构（IVF/IVF+PQ/HNSW 等索引占用额外存储。PQ 可以压缩向量。）、元数据/时间戳/分区信息（小于总存储量 10%，可忽略。）
高维向量+大规模数据集，需要考虑索引压缩和分片策略(sharding/segment)。
分配策略：是对Collection数据的一种逻辑划分方式。（水平扩展、负载均衡、并行计算）
|Collection|用户层逻辑表|包含多个Partition|
|Partition|Collection下的子集|支持逻辑隔离与并行查询|
|segment|数据的最小存储单位|每个segment存储一定数量的向量，并分配到DataNode/QueryNode|
|replica|segment的副本|提高可用性和容错能力|
segment类型：sealed segment 封存段（只读，已完成写入和索引构建）、growing segment 增长段（可写，用于实时插入数据）
分片策略实现
（1）基于 Segment 的自动分片
写入数据：客户端插入向量到 Collection。DataCoord 分配数据到 Growing Segment。当 Segment 达到阈值（例如默认 1M 行），DataCoord 封存该 Segment。
查询数据：QueryCoord 获取 Collection 下所有 Segment 信息（哪些节点存哪些 Segment）。QueryNode 并行扫描分配给自己的 Segment。Proxy 聚合返回结果给客户端。
（2）replica分配策略
Segment 可以有多个副本（Replica），分布在不同节点上。写入和查询请求可以在副本之间负载均衡。当节点挂掉时，其他副本继续服务，实现高可用。
（3）分片迁移策略
新节点加入集群：Coord会重新计算Segment分配，迁移部分Segment到新节点。查询负载随节点增加而均衡分布。
节点下线时：Segment 副本自动迁移到其他节点。
| 方面     | 影响 | 说明                                    |
| ------ | -- | ------------------------------------- |
| 并行查询   | 高  | Segment 分布在多个 QueryNode，可并行处理         |
| 数据写入   | 高  | Growing Segment 分布在多个 DataNode，支持并行写入 |
| 负载均衡   | 高  | Coord 根据节点资源动态分配 Segment              |
| 数据迁移成本 | 中  | Segment 太大时迁移开销大，需要控制 Segment 大小      |
| 容错性    | 高  | Replica 副本保证节点故障不丢数据                  |
分片策略实践建议：
（1）segment大小调优：默认 1M 行，可根据业务写入速率和查询性能调整。太小 → 分片过多，管理开销大；太大 → 数据迁移慢。
（2）节点资源匹配：高维向量 + 大规模数据 → 节点 CPU、内存要充足。
（3）replica数量：根据容错需求设置，一般 2~3 个副本。
（4）动态扩展：利用 Kubernetes 或容器管理，按负载自动增加 QueryNode/DataNode。
（5）索引与分片结合：索引分布在 Segment 内，每个 Segment 可独立建立索引。索引选择会影响 Segment 查询性能。
实践：导入 1 万条向量，测试搜索耗时

6、用 Python SDK 实现一个“向量数据库最小示例”

7、复盘 CRUD、Schema、基本检索
思考：Milvus 和传统数据库（如mysql）的区别
| 对比维度       | Milvus                                                   | MySQL                                      |
| ---------- | -------------------------------------------------------- | ------------------------------------------ |
| **定位**     | 向量数据库（Vector Database），专门用于高维向量数据存储和相似度搜索。| 关系型数据库（RDBMS），用于结构化数据存储和事务处理。|
| **数据类型**   | 支持高维向量（如图像特征、文本嵌入向量） + 基本属性字段（数值、字符串等）。| 支持结构化表格数据（整数、浮点数、字符串、日期等）。|
| **存储方式**   | 使用向量索引（如 IVF、HNSW、PQ 等）优化相似度搜索，通常配合分布式存储（MinIO、RocksDB）。| 基于 B+ 树、哈希索引等存储结构优化主键查询、范围查询。|
| **查询类型**   | 主要是相似度搜索（kNN）、向量距离计算（欧氏、余弦、内积），支持海量高维向量检索。| SQL 查询为主，支持事务、复杂 JOIN、聚合、条件筛选等操作。|
| **扩展性**    | 天生支持水平扩展，可通过多副本、多分区、多计算节点扩展存储与计算能力。| 水平扩展较复杂，通常通过分片或读写分离实现，更多依赖外部中间件。|
| **性能特点**   | 针对高维向量搜索进行了优化，支持千万到亿级向量快速检索，低延迟。| 针对 OLTP 或 OLAP 场景优化，高并发事务处理能力强，但高维向量搜索效率低。 |
| **典型应用场景** | 图像搜索、语义搜索、推荐系统、自然语言处理 embeddings。| 企业业务系统、财务系统、ERP、用户信息管理等结构化数据处理。|
| **事务与一致性** | Milvus 不强调强事务（ACID），主要关注向量写入和搜索性能。| MySQL 提供 ACID 事务，保证数据强一致性。|
总结：
MySQL 是通用关系型数据库，擅长结构化数据存储、事务处理。
Milvus 是专门为向量数据设计的数据库，擅长高维向量检索和相似度搜索，两者在底层设计、查询方式和应用场景上完全不同。

8、近似最近邻（ANN）的基本原理
基本思想：通过索引结构，把向量空间划分为若干子区域，避免对所有点做距离计算，只在候选集合中搜索最近邻。
常见方法：
（1）基于量化：把高维向量压缩成低维或离散化表示，减少计算量。如PQ、IVF+PQ
（2）基于图：构建一个近邻图（如 kNN Graph），点与点之间建立边，搜索时沿着图走。如HNSW
（3）基于树或空间划分：递归划分空间（类似 kd-tree, ball-tree），减少搜索空间。如Annoy、LSH
搜索过程：（以 IVF+PQ 为例）
（1）建索引：对向量做 k-means 聚类，得到若干“中心点”（簇）。每个向量归入最近的簇，并在簇内做 PQ 压缩存储。
（2）查询：对查询向量𝑞，先找到最近的几个簇（比如 top-10 簇）。在这些簇里，用 PQ 码本快速估算距离，筛选候选集。对候选集做精确计算，得到最终最近邻。
实践：创建 IVF 索引

9、nlist、nprobe 参数的意义 ✅
向量集合先经过 k-means 聚类，划分成若干 簇，簇的数量就是nlist。查询时，先把查询向量𝑞与所有簇中心比对，找到最近的 nprobe 个簇。只在这nprobe个簇内搜索候选向量，而不是全量搜索。
nlist调整效果：
- nlist 大：划分更细，每个簇的向量更少 → 检索更快（因为簇内数据量小），但聚类偏差可能更大，需要合适的 nprobe 来补偿。
- nlist 小：划分更粗，每个簇数据量大 → 检索可能慢，但簇内更容易找到真正邻居。
一般经验：nlist ≈ sqrt(N) （N = 数据量），是经验上比较合适的范围。
nprobe调整效果：
- nprobe 大：覆盖簇更多，召回率更高（更可能找到真正最近邻），但计算开销增加 → 查询更慢。
- nprobe 小：只看少量簇，速度快，但可能漏掉真正最近邻 → 召回率下降。
调优建议：
- 数据量 小（< 1M）：nlist 不宜过大（几百上千即可），nprobe 可以适中（520）。
- 数据量 大（> 10M）：nlist 可以上万，nprobe 需要调节（几十~几百），在 速度和召回率 之间找平衡。
一般做法：1、先固定合适的 nlist（训练时决定）。2、在查询时调整 nprobe，根据业务需求权衡 速度 vs. 准确率。
实践：调优 IVF 参数，比较延迟与准确率 ✅
（1）先确定 nlist（建索引时固定）
推荐用sqrt(N)级别的值，然后在 2–3 个数量级范围内测试。
（2）再调节 nprobe（查询时灵活调）
小步调优，画出 recall–latency 曲线，找到拐点。
（3）业务导向
推荐系统 → recall ≥ 0.95，延迟 50ms 内可接受。
实时搜索 → recall ≥ 0.85 即可，延迟 < 10ms。

10、HNSW 算法（ef、M 参数）
HNSW 本质上是一个多层近邻图：上层稀疏，下层稠密，查询时从上层“粗定位”，逐层往下refine，直到最底层找到候选近邻。在底层，图的连边保证局部接近“小世界性质”（任何点之间距离不会太远），搜索效率高。
参数 M：
- 定义：每个节点在图中最多连接的邻居数量（出度上限）。
- 影响：M 大 → 图更稠密 → 搜索精度高，但索引更大、建图更慢、内存占用更高。M 小 → 图更稀疏 → 内存占用少，索引快，但搜索可能掉精度。
- 经验值：通常在 5 ~ 64 之间。一般推荐 M = 16 或 32（FAISS、Milvus 默认值差不多）。
参数 ef：
(1) efConstruction
用于 建图阶段。
控制在建立节点连接时，候选集合的大小。
大 efConstruction → 建图更慢、更耗内存，但图的质量更高（召回率更好）。
常见设置：100 ~ 200，大规模数据可以更大。
(2) efSearch
用于 查询阶段。
控制在搜索时维护的候选节点集合大小。
大 efSearch → 查询时探索更多节点 → 准确率更高，但延迟增加。
小 efSearch → 查询更快，但 recall 降低。
通常取值范围：efSearch ≈ k × 2 ~ 10（k = Top-K 的数量）。例如 Top-10 检索，efSearch = 50 ~ 200 是常见区间。
实际调优方法：
- 固定 M（一般 16 或 32）。
- 建索引时调 efConstruction：小数据可取 100–200；大数据可取 300–500。
- 查询时调 efSearch：从较小值开始（如 50），逐渐增大，观察 recall–latency 曲线。找到 recall 提升趋缓的拐点（再加大只增加延迟，不显著提高准确率）。
实践：创建 HNSW 索引并搜索

11、对比实验：IVF vs HNSW vs Flat ✅
模拟结果：
===== 对比结果（N=100000, DIM=128, NQ=200, k=10) =====
Index    Avg Latency(ms) QPS        Recall@10 
FLAT     108.24         9.24       1.00(基线)  
IVF      1.60           623.52     0.2825    
HNSW     1.83           547.19     0.6290 
输出：延迟 + Recall 对比表

12、索引构建开销（内存、磁盘）
实践：测量不同索引的内存占用

13、用 Annoy 或 DiskANN 索引（如支持）
DiskANN: 
- Milvus 原生支持 DiskANN 索引类型，这是一个基于 Vamana 图的图索引，专为 大规模向量数据设计.
- 使用 SSD 存储索引，大幅降低 RAM 占用，适用于亿级乃至十亿级向量数据 —— 即使总数据量超过内存，依然能提供较低延迟（如几十 ms）和高召回（如 ~95%）。
- DiskANN 特定参数可在 Milvus 配置文件（milvus.yaml）中设定，如：MaxDegree、SearchListSize、PQCodeBudgetGBRatio、BeamWidthRatio 等。
- 如果你需要在内存有限的机器上处理海量向量，DiskANN 是 Milvus 官方推荐且支持的磁盘索引方案。
Annoy：
- Annoy 是一种基于多棵树构建的向量索引结构，适合静态数据和资源受限场景。
- Java SDK 目前不支持 Annoy 类型。
思考：适合大规模数据的索引策略

14、整理索引对比表（性能、适用场景）
| 索引类型           | Milvus Java SDK 支持 | 内存需求     | 适用场景                |
| -------------- | ------------------ | -------- | ------------------- |
| **DiskANN**    | 是                  | 低，依赖 SSD | 海量向量、内存受限时的高效 ANN   |
| **Annoy**      | 否                  | 低        | 静态数据、资源受限，但 **不推荐** |
| **HNSW / IVF** | 支持                 | 较高       | 内存充裕、需要高准确率查询       |


15、RocksDB 原理（LSM-Tree、WAL）
（1）RocksDB 概述
RocksDB 是 Facebook 基于 LevelDB 开发的嵌入式 KV 存储引擎。
特点：高写入吞吐、低延迟、良好的压缩机制，适合 SSD 场景。
核心原理：基于 LSM-Tree（Log-Structured Merge-Tree） 数据结构。
（2）LSM-Tree（Log-Structured Merge-Tree）
LSM-Tree 是一种 写优化的存储结构，其思路是：a.写入先落内存（MemTable），再批量刷盘，避免随机写磁盘。b.通过多层排序文件（SSTable）+ Compaction 保证读写性能。
RocksDB 的 LSM-Tree 流程：
- 写入：a.写请求首先写到 WAL（Write-Ahead Log），保证宕机恢复。b.同时写入内存结构 MemTable（跳表/红黑树，保持有序）。
- MemTable → SSTable: MemTable 满了以后，转为 Immutable MemTable，后台刷盘生成 SSTable 文件（有序的 KV 存储文件）。
- 多层 SSTables（Level-0, Level-1, …）: RocksDB 按层次存放 SSTables：Level-0：新写入，文件较少，但键范围可能重叠。Level-1 及以上：有序、范围不重叠。
- Compaction（压缩/合并）: 后台线程不断将高层文件合并到低层（如 L0 → L1），消除重复键、回收空间。代价：读放大、写放大、空间放大，需要调优。
（3）WAL（Write-Ahead Log）- WAL 是 RocksDB 数据持久化保证的核心机制。
原理：
- 每次写入（Put/Delete）：先写一条记录到 WAL 文件（顺序写，持久化到磁盘）。再更新内存里的 MemTable。
- 如果数据库崩溃：下次重启时，RocksDB 会 重放 WAL，恢复 MemTable，保证不丢数据。
优点：
- 避免宕机导致数据丢失。
- 顺序写磁盘，效率高。
配置优化：
- 可以选择 异步 fsync，在性能和可靠性之间平衡。
- 大多数场景：WAL 打开 + 定期 flush → 能兼顾性能和安全。
（4）RocksDB 的特点
- 优点：写性能高（顺序写 + 批量落盘），适合写多读少场景（如日志、消息队列、推荐系统 embedding 存储）。
- 缺点：读可能涉及多层查找（读放大），需要 Bloom Filter、Block Cache 辅助。
实践：查看 Milvus 元数据写入 RocksDB

16、S3/MinIO 对象存储
MinIO：MinIO 是一个高性能、开源的 对象存储系统，兼容 Amazon S3 API。可以把它看作“私有化 S3”，你可以在本地机房、容器环境、K8s 中部署 MinIO，像用 S3 一样访问对象存储。大文件存储、AI/大数据/机器学习训练数据存储、数据库索引文件存储（如 Milvus）、日志归档、镜像仓库后端等。
MinIO console：http://127.0.0.1:9001/browser  minioadmin
Milvus将segment、索引存储到MinIO
实践：部署 MinIO 并配置 Milvus


17、Milvus 如何用 RocksDB + MinIO 组合存储
元数据（collection、partition、segment 状态、索引任务等）—— RocksDB
向量原始数据 & 索引文件 —— MinIO
配置文件位置：/mivlus/configs/mivlus.yaml
实践：插入 100 万向量，观察数据落盘情况

18、Partition、Sharding 的意义
| 特性| Partition（分区）| Sharding（分片）|
| ------ | ---------------- | -------------- |
| 定义层面| 逻辑层（用户可见）| 物理层（系统内部）|
| 控制方式| 用户手动创建/选择| 系统自动管理|
| 目的| 减少搜索空间、提高查询效率| 支持分布式存储和计算，扩展性 |
| 是否用户感知 | ✅ 用户可见（创建、查询时指定）| ❌ 用户透明，不可直接操作|
| 类比| 数据库表分区| 数据库分库分表|
Partition：在一个 Collection 里按条件（比如标签、业务线、时间范围）划分数据，方便查询时只搜一部分数据。
sharding：当一个 Collection 或 Partition 里的数据太多，Milvus 会把数据切分成多个 Segment，分布到不同的节点（Shard）上。
实践：创建 Partition 并做分区检索 ✅

19、Milvus 集群架构
（1）Milvus 集群总体架构
Milvus 采用 存算分离 + 分层组件的设计，主要包括四层：
┌─────────────────────────┐
│        Client API        │
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│         Proxy            │
│  （接入层，请求路由）    │
└───────────┬─────────────┘
            │
 ┌──────────▼───────────┐
 │      RootCoord        │
 │      QueryCoord       │
 │      DataCoord        │   （协调层）
 └──────┬──────┬────────┘
        │      │
┌───────▼───┐  ┌─────────▼─────────┐
│  DataNode │  │    QueryNode       │  （执行层）
└───────────┘  └───────────────────┘
        │               │
        ▼               ▼
 ┌───────────────┐ ┌───────────────┐
 │   ObjectStore │ │  Metadata DB  │
 │ (MinIO / S3)  │ │ RocksDB/MySQL │
 └───────────────┘ └───────────────┘
（2）各组件作用
A. Proxy（接入层）
- 所有客户端请求先到 Proxy
- 功能：请求校验（collection 是否存在、参数合法性）、路由请求给合适的 Coordinator、聚合不同节点的结果并返回
B. Coordinator（协调层）
协调层有三个角色：
- RootCoord：管理 元数据（collection/partition/schema）、分配 ID、维护时间戳逻辑时钟（保证一致性）
- DataCoord：负责 数据写入管理、把插入的数据打包成 Segment，调度到 DataNode、触发 Flush，把 Segment 持久化到对象存储（MinIO）、触发 Compaction（合并小 Segment）
- QueryCoord：负责 查询调度、管理哪些 Segment 被加载到哪些 QueryNode、负责负载均衡（避免单点 QueryNode 过载）
C. Worker 节点（执行层）
- DataNode：负责 写数据：接收 Insert/Delete 流、将数据写入内存，刷盘时写到对象存储（MinIO）、维护增量日志（delta log）
- QueryNode：负责 读数据：执行向量检索、标量过滤；从对象存储拉取 Segment & 索引文件加载到内存；执行向量检索算法（IVF/HNSW/Flat 等）
D. 存储层
- 对象存储（MinIO/S3/Ceph/OSS …）:存放 Segment 文件、索引文件/数据量大，支持无限扩展
- 元数据存储（RocksDB/MySQL/Postgres）:存储 collection、partition、segment、索引等元信息；RocksDB 默认（单机），生产建议 MySQL/Postgres（支持高可用）
（3）数据写入流程
客户端 Insert
   ↓
Proxy（校验、路由）
   ↓
RootCoord 分配 ID
   ↓
DataCoord 调度 Segment
   ↓
DataNode 接收数据 → 内存缓冲
   ↓
Flush → Segment 文件写入 MinIO
   ↓
元数据（segment 状态）写入 RocksDB/MySQL
（4）数据查询流程
客户端 Search/Query
   ↓
Proxy（解析、路由）
   ↓
QueryCoord 决定由哪些 QueryNode 执行
   ↓
QueryNode 加载 Segment & 索引文件（从 MinIO）
   ↓
向量检索（IVF/HNSW/Flat）
   ↓
聚合结果，返回给 Proxy
   ↓
客户端获取结果
（5）架构特点
- 存算分离：计算节点（DataNode/QueryNode）和存储（MinIO）解耦，可独立扩展
- 弹性扩展：数据大了可以加节点，QueryNode 横向扩展提高查询吞吐
- 高可用：Coordinator 节点可多副本部署（K8s + etcd 协调）
- 统一存储：向量 + 标量数据一体化，索引和原始数据都存对象存储
- 支持分区（Partition）+ 分片（Shard），提升查询效率与扩展性
实践：在 K8s 部署 Milvus（Operator）

20、模拟节点宕机，观察数据恢复过程
思考：Milvus 的一致性保障机制

21、画一张“Milvus 数据存储与架构流程图”

22、RAG（检索增强生成）原理

实践：Milvus + OpenAI Embedding，存入文本向量



























