好的，我会在你之前的 **30 天学习计划** 基础上，把每天的**学习内容（理论）**和**项目设计内容（实作）**详细展开，形成一个你可以**照单执行**的落地指南。
我们按**每天：学习目标 → 学习内容 → 项目设计内容 → 输出物**的结构来写，这样能让你同时兼顾理论和工程实现。

---

## **第 1 周（Day 1–7） 基础与 API 集成**

### **Day 1 — LLM 基础与能力边界**

* **学习目标**：理解 LLM 的工作方式和限制，为后续 Agent 架构设计打基础。
* **学习内容**：

  * Transformer 架构基本原理（注意力机制、tokenization、上下文窗口）。
  * 参数：temperature、top\_p、max\_tokens 的作用与取值。
  * LLM 在推理、事实性、生成上的能力与局限。
  * 常见商用 API（OpenAI、Anthropic、DeepSeek、Gemini）的调用模式对比。
* **项目设计内容**：

  * 创建 Java Maven/Spring Boot 项目，规划模块目录：

    ```
    com.example.agent
        ├── llm (模型调用)
        ├── tools (外部工具)
        ├── memory (上下文与记忆)
        ├── rag (检索增强)
        ├── planner (任务规划)
        ├── executor (任务执行)
    ```
  * 在 `llm` 模块中创建第一个 LLM API 客户端（直接用 REST 调用）。
* **输出物**：

  * 一张 LLM 工作原理流程图。
  * 一个能在 `main()` 中调用 LLM API 并输出文本的 Java 类。

---

### **Day 2 — Prompt 工程与对话管理**

* **学习内容**：

  * Prompt 模板设计方法（system prompt / user prompt / assistant prompt）。
  * Few-shot、Chain-of-Thought、思维链和 ReAct Prompt 风格。
  * Token 计算与上下文截断策略（OpenAI Tokenizer 工具）。
* **项目设计内容**：

  * 实现 `PromptTemplate` 类，支持：

    ```java
    PromptTemplate template = new PromptTemplate("You are a helpful assistant. Answer in JSON: {{question}}");
    String prompt = template.fill(Map.of("question", "What is Java?"));
    ```
  * 支持 token 计数和上下文裁剪方法。
* **输出物**：

  * 5 个不同任务的 prompt 模板文件（JSON / YAML）。
  * 运行示例：输入问题，模板填充后发给 LLM，得到输出。

---

### **Day 3 — Java 与 LLM 的最佳实践（HTTP、Streaming、重试）**

* **学习内容**：

  * HTTP 长连接与流式响应（SSE）。
  * Resilience4j 的重试、熔断、超时配置。
  * Reactive 编程在 Streaming 输出中的应用（WebFlux Flux/SSE）。
* **项目设计内容**：

  * 用 WebClient 实现 streaming 消费：

    ```java
    webClient.post()
        .uri("/v1/chat/completions")
        .bodyValue(request)
        .retrieve()
        .bodyToFlux(String.class)
        .subscribe(System.out::print);
    ```
  * 在调用层加 Resilience4j 的超时和重试策略。
* **输出物**：

  * LLM 流式输出 Demo（能实时打印 token）。
  * 重试策略配置文件。

---

### **Day 4 — 函数调用 / 工具调用抽象**

* **学习内容**：

  * LLM function\_call / tool invocation 概念。
  * Java 接口设计原则（接口隔离、单一职责）。
  * JSON Schema 校验（防止 LLM 返回脏数据）。
* **项目设计内容**：

  * 定义 Tool 接口：

    ```java
    public interface Tool<I, O> {
        String name();
        O execute(I input);
    }
    ```
  * 实现两个工具：

    * CalculatorTool（简单加减乘除）。
    * SearchTool（调用一个免费的搜索 API）。
  * 写一个 `ToolRegistry` 管理工具注册与调用。
* **输出物**：

  * 能由 LLM 触发调用 Java 工具并返回结果的 Demo。

---

### **Day 5 — 对话上下文与短期记忆**

* **学习内容**：

  * 短期记忆：当前对话 session 的上下文管理。
  * 会话摘要：旧消息压缩成摘要。
  * 存储方案（内存 / 数据库）。
* **项目设计内容**：

  * 实现 `SessionMemory` 类，存储最近 N 条消息。
  * 当消息总 token 数 > 阈值时，调用 LLM 生成摘要替换旧消息。
* **输出物**：

  * 能在多轮对话中保持上下文的 REST API。

---

### **Day 6 — 小练习：构建一个问答式 Agent**

* **学习内容**：

  * Agent 的基本循环：输入 → LLM → 工具调用（可选）→ 输出。
* **项目设计内容**：

  * 在 Spring Boot 中实现 `/chat` 接口：

    * 输入：用户问题。
    * 调用：LLM API + 工具调用。
    * 输出：模型回复。
* **输出物**：

  * 可运行的最小 Agent Demo（curl 请求即可得到回答）。

---

### **Day 7 — 回顾与扩展计划**

* **学习内容**：

  * 对已完成的模块进行代码审查。
  * 梳理向量数据库、embedding、RAG 的基本概念。
* **项目设计内容**：

  * 在项目中预留 `rag` 模块。
  * 编写后续需要的第三方依赖清单（向量 DB SDK、PDF 解析库等）。
* **输出物**：

  * 周总结文档（技术点 + 遇到的问题 + 下一步计划）。

---

## **第 2 周（Day 8–14） RAG（检索增强生成）与长短期记忆**

---

### **Day 8 — 文本预处理与 Chunking 策略**

* **学习目标**：能将原始文档分割成适合向量化的片段。
* **学习内容**：

  * 文本切分策略：固定长度、滑动窗口（overlap）、语义分句。
  * 选择合适 chunk size（200–500 tokens 常用）。
  * 处理多种格式（Markdown、PDF、HTML）。
* **项目设计内容**：

  * 在 `rag` 模块创建 `TextChunker` 类：

    ```java
    public List<String> chunk(String text, int size, int overlap)
    ```
  * 集成 Apache PDFBox / Jsoup 提取文本。
  * 输出带 metadata 的 chunk（文档 ID、页码、段落位置）。
* **输出物**：

  * 从一份 PDF 文档生成 chunk JSON 的示例。

---

### **Day 9 — Embeddings 与向量存储**

* **学习目标**：掌握将文本转换为向量并存储的流程。
* **学习内容**：

  * Embedding API 的原理与调用方式。
  * 常见距离度量（cosine、dot、L2）。
  * 向量数据库（Milvus、Weaviate、Pinecone）基础概念。
* **项目设计内容**：

  * 创建 `EmbeddingClient` 接口：

    ```java
    public interface EmbeddingClient {
        List<Float> embed(String text);
    }
    ```
  * 实现 OpenAI Embeddings 调用。
  * 写向量数据库 HTTP 客户端，支持 upsert、search。
* **输出物**：

  * 能将 Day 8 的 chunk 全部向量化并写入向量库。

---

### **Day 10 — 向量检索集成**

* **学习目标**：将检索功能集成到 Java 项目中。
* **学习内容**：

  * 向量搜索流程：输入 → embedding → top-k 检索。
  * 多字段检索：metadata filter。
* **项目设计内容**：

  * 实现 `VectorStoreRetriever`：

    ```java
    public List<Document> retrieve(String query, int topK);
    ```
  * 支持 metadata filter（如只检索特定文档）。
* **输出物**：

  * 给定查询，能返回最相关的文档片段（打印来源信息）。

---

### **Day 11 — RAG Pipeline**

* **学习目标**：实现完整的检索+生成流程。
* **学习内容**：

  * context 拼接策略（按相关度排序）。
  * prompt 模板设计（加来源、加引用标记）。
* **项目设计内容**：

  * 在 `rag` 模块创建 `RAGOrchestrator`：

    * 调用 retriever 获取片段。
    * 拼接到 prompt。
    * 调用 LLM 生成答案。
  * 输出带引用的回答。
* **输出物**：

  * 输入任意问题，返回带 \[source\_x] 引用的回答。

---

### **Day 12 — 长期记忆（Long-term Memory）**

* **学习目标**：设计并实现对历史对话的长期存储与检索。
* **学习内容**：

  * 何时存储对话（事件驱动 / 阈值）。
  * 长期记忆检索优先级（最新优先 / 语义优先）。
* **项目设计内容**：

  * 在 `memory` 模块创建 `LongTermMemory`：

    * 将重要对话向量化后存入向量库。
    * 检索时合并短期记忆和长期记忆结果。
* **输出物**：

  * 能跨会话回答历史对话的问题。

---

### **Day 13 — 性能与成本分析（RAG）**

* **学习目标**：衡量 RAG 系统的延迟与成本。
* **学习内容**：

  * Embedding API 成本计算方法。
  * 检索延迟优化（batch query、缓存）。
* **项目设计内容**：

  * 编写 Benchmark 脚本：

    * 插入 10k 条向量。
    * 测量单次查询延迟与吞吐量。
  * 测试缓存命中率对延迟的影响。
* **输出物**：

  * 性能分析报告（延迟、成本、优化建议）。

---

### **Day 14 — RAG MVP 演示**

* **学习目标**：整合本周成果为可演示系统。
* **项目设计内容**：

  * 用 REST API `/rag/query`：

    * 输入：自然语言问题。
    * 流程：检索 → 拼接 → 生成。
  * 支持返回引用文档和置信度分数。
* **输出物**：

  * 录制一个 Demo（问答 + 引用来源）。

---

## **第 3 周（Day 15–21） Planner 与 Executor（多步 Agent）**

---

### **Day 15 — Agent 架构设计**

* **学习目标**：理解 Planner、Executor、Memory、Tools 的协作。
* **学习内容**：

  * 中央编排（单 Planner） vs 分布式 Agents。
  * 数据流设计（任务队列、状态流转）。
* **项目设计内容**：

  * 画架构图（组件+数据流）。
  * 在代码中定义：

    ```java
    interface Planner { List<Task> plan(String goal); }
    interface Executor { TaskResult execute(Task task); }
    ```
* **输出物**：

  * 架构图 + 接口定义代码。

---

### **Day 16 — 任务分解（CoT / ReAct）**

* **学习内容**：

  * Chain-of-Thought 与 ReAct 的差异。
  * JSON Schema 限制 LLM 输出格式。
* **项目设计内容**：

  * 实现 `LLMPlanner`：

    * 接收目标任务，生成 JSON 格式的子任务列表。
    * 校验 JSON 结构（用 Jackson + JSON Schema）。
* **输出物**：

  * 输入任务“收集 Java 新闻并总结”，Planner 输出 3–5 个步骤。

---

### **Day 17 — Executor 并发与回滚**

* **学习内容**：

  * 并发执行（线程池、Reactor Flux）。
  * 回滚/补偿机制设计。
* **项目设计内容**：

  * 在 `executor` 模块实现：

    * 并行执行任务（带超时、重试）。
    * 对失败任务调用补偿逻辑。
* **输出物**：

  * 演示多任务并发执行，并在失败时回滚。

---

### **Day 18 — 分布式任务编排**

* **学习内容**：

  * Kafka/RabbitMQ 在 Agent 调度中的作用。
  * 事件驱动架构（EDA）。
* **项目设计内容**：

  * 用 Kafka topic 传递 Task 和 Result。
  * 写一个 Consumer 处理任务，一个 Producer 发送任务。
* **输出物**：

  * 能通过 Kafka 分发任务到多个 Executor 节点。

---

### **Day 19 — 人类在环（HITL）与审计**

* **学习内容**：

  * HITL 的场景与设计（人工确认）。
  * 审计日志（可追溯性）。
* **项目设计内容**：

  * 为敏感任务加入人工确认接口 `/approve/{taskId}`。
  * 审计日志存储（DB + JSON 格式）。
* **输出物**：

  * 演示任务暂停等待人工批准后继续。

---

### **Day 20 — 工具安全与隔离**

* **学习内容**：

  * 工具调用白名单、参数校验。
  * 最小权限执行（sandbox）。
* **项目设计内容**：

  * 在 Tool 调用前加输入校验和执行时间限制。
  * 对外部 API 调用加速率限制。
* **输出物**：

  * 触发恶意输入时拒绝执行的示例。

---

### **Day 21 — 集成测试与多场景演练**

* **学习内容**：

  * 集成测试覆盖率设计。
  * 场景测试（正常 / 异常 / 恶意输入）。
* **项目设计内容**：

  * 写集成测试模拟：

    1. 简单任务。
    2. 带工具调用任务。
    3. 跨多个 Executor 的任务。
* **输出物**：

  * 测试报告（通过率、耗时）。

---

## **第 4 周（Day 22–28） 生产化、监控、评估与部署**

---

### **Day 22 — 可观测性**

* **学习内容**：

  * OpenTelemetry、Micrometer、Prometheus、Grafana。
  * 分布式追踪。
* **项目设计内容**：

  * 给 LLM 调用、工具调用加指标收集（延迟、token 数、失败率）。
  * 集成 Jaeger 做调用链追踪。
* **输出物**：

  * Grafana Dashboard 展示实时指标。

---

### **Day 23 — 可靠性工程（SLO、限流、降级）**

* **学习内容**：

  * SLO/SLA 定义。
  * 限流算法（令牌桶）、降级策略。
* **项目设计内容**：

  * 对外部 API 调用加 Resilience4j 限流。
  * 在降级时返回缓存数据或简化模型输出。
* **输出物**：

  * 压测下自动降级的演示。

---

### **Day 24 — 测试与评估**

* **学习内容**：

  * LLM 质量评估（准确率、幻觉率）。
  * 对抗测试（prompt injection）。
* **项目设计内容**：

  * 编写自动化评估脚本，批量输入测试集，统计指标。
  * 写几条 prompt injection 测试。
* **输出物**：

  * 评估报告（含防御策略）。

---

### **Day 25 — 性能优化**

* **学习内容**：

  * 批量处理、缓存、模型路由。
* **项目设计内容**：

  * Embeddings 批量请求。
  * Query 缓存（LRU + TTL）。
  * 简单任务用小模型，大任务用大模型。
* **输出物**：

  * 优化前后性能对比表。

---

### **Day 26 — 部署（Docker / K8s / CI）**

* **学习内容**：

  * Docker 多阶段构建。
  * Kubernetes Deployment + HPA。
* **项目设计内容**：

  * 写 Dockerfile、k8s manifest（deployment/service/configmap/secret）。
  * 配置 GitHub Actions 自动构建推送镜像。
* **输出物**：

  * 部署在 k8s 集群并可访问的服务。

---

### **Day 27 — 数据治理与合规**

* **学习内容**：

  * PII 检测、数据脱敏。
  * GDPR/CCPA 基础。
* **项目设计内容**：

  * 实现敏感数据检测正则。
  * 用户数据删除 API。
* **输出物**：

  * 对包含手机号/邮箱的输入自动脱敏的示例。

---

### **Day 28 — 成本模型**

* **学习内容**：

  * LLM 调用成本计算（token 计费）。
  * 成本优化方法。
* **项目设计内容**：

  * 写成本统计模块（记录调用次数、token 数、费用）。
  * 生成成本月报。
* **输出物**：

  * 成本分析表 + 优化建议。

---

## **Day 29 — 文档与 Demo**

* **学习内容**：

  * 架构文档编写规范（C4 模型）。
  * Demo 脚本设计。
* **项目设计内容**：

  * 写 README、架构图、API 文档（Swagger）。
  * 准备 10 分钟演示视频脚本。
* **输出物**：

  * 完整文档 + 演示视频。

---

## **Day 30 — 回顾与路线图**

* **学习内容**：

  * 技术债整理。
  * 新功能规划。
* **项目设计内容**：

  * 制定 90 天路线图（优先级排序）。
  * 评估是否引入 Fine-tuning / 多 Agent 协作。
* **输出物**：

  * 路线图文档（含时间表与负责人分配建议）。

---



---

## AI Agent 系统 —— 架构图文字说明

```
                                     ┌──────────────┐
                                     │ External API │
                                     │ (e.g., Web)  │
                                     └──────┬───────┘
                                            │
                                       Incoming User
                                            │
                                      ┌─────▼─────┐
                                      │  REST API │
                                      │ (Spring)  │
                                      └─────┬─────┘
                                            │
                                +-----------▼-----------+
                                │  Short-term Memory     │
                                │ (Session Store +       │
                                │  Context Summarizer)   │
                                +-----------┬-----------+
                                            │
                                +-----------▼------------+
                                │     Planner            │
                                │ (LLM-based task        │
                                │  decomposition)        │
                                +-----------┬------------+
                                            │ produces task graph
                                +-----------▼------------+
                                │     Executor            │
                                │  (Scheduler + Dispatcher│
                                │   + Tool Runner Pool)    │
                                +----┬--------┬----------+
                                     │        │
                  ┌──────────────────▼┐       │
                  │ Tool Invocation    │       │
                  │ (Search, Calculator,│       │
                  │ HTTP, Database, etc)│       │
                  └──┬───────────────┬──┘       │
                     │               │          │
            returns result        error/fallback │
                     │               │          │
          ┌──────────▼┐     ┌────────▼──────┐     │
          │ RAG Retriever│   │ Long-term     │     │
          │ (Vector DB)  │   │ Memory Store  │     │
          └─────┬───────┘   └─────┬────────┘     │
                │                 │              │
                └───────┬─────────┘              │
                        │ raises context chunks  │
                ┌───────▼─────────────────────┐   │
                │  RAG Augmented Context     │   │
                │ (prompt + retrieved chunks)│←──┘
                └───────┬─────────────────────┘
                        │
                 ┌──────▼───────┐
                 │    LLM       │
                 │ (Completion /│
                 │  function_call│
                 │   streaming)  │
                 └──────┬────────┘
                        │ response + tool call?
                        │
                   ┌────▼────┐
                   │Post-process│
                   │(parse JSON,│
                   │ validate,   │
                   │ log audit)  │
                   └────┬───────┘
                        │
                  ┌─────▼─────┐
                  │  REST API │ replies to user
                  └─────┬─────┘
                        │
                ┌───────▼────────┐
                │  Observability │
                │ (Metrics /      │
                │  Tracing /      │
                │  Logging / Audit)│
                └──────┬─────────┘
                        │
       ┌───────────────┬───┴───┬───────────────┐
       │               │       │               │
Monitoring Dashboards  Logs Traces           Alerts
( Grafana, etc.)      (Elasticsearch, etc.) ( PagerDuty, etc.)
```

---

## 模块说明（图中各部分对应解释）

| 模块名                   | 功能说明                                  |
| --------------------- | ------------------------------------- |
| **REST API**          | 接收用户请求（如通过 HTTP），并返回响应。所有 Agent 操作入口。 |
| **Short-term Memory** | 在 session 中管理对话上下文，并负责生成摘要减少 token 数。 |
| **Planner**           | 使用 LLM 将用户目标拆解成结构化任务列表（JSON、动作序列）。    |
| **Executor**          | 调度并执行拆解后的任务，管理工具调用、并发、错误处理与补偿。        |
| **Tool Invocation**   | 封装外部工具（搜索、计算器、HTTP 请求、数据库操作等）调用逻辑。    |
| **RAG Retriever**     | 从向量数据库中检索与用户上下文相关内容，作为生成 prompt 的输入。  |
| **Long-term Memory**  | 存储经过筛选的重要对话或事件，供跨会话记忆检索使用。            |
| **LLM**               | 中央生成引擎，执行生成和/或函数调用、流式输出。              |
| **Post-process**      | 解析和验证 LLM 返回结果（例如 JSON），执行审计日志写入。     |
| **Observability**     | 收集指标、追踪调用链，便于监控、告警和性能分析。              |

---

## 如何绘制这张图

1. **选择在线作图工具**，如 draw\.io、Excalidraw 或 Lucidchart。
2. **用矩形表示系统组件**，用箭头表示数据或调用流。
3. **不同颜色区分**：

   * Web 接口/用户交互（REST API）。
   * LLM 与任务逻辑（Planner、Executor、Tools）。
   * 存储（RAG Retriever、Memory）。
   * 观测和日志（Observability）。
4. **在每个模块内标注接口或关键函数名称**，如 `Planner.plan(goal) → List<Task>`、`Executor.execute(task)`。

---
