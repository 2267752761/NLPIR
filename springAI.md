Chat Client API
advisor


function type:
Function<T:
 R>: 将一个类型的值转换为另一个类型
Supplier: 提供一个类型的值，不接受参数
Consumer: 接收一个输入值进行处理，不返回结果
BiFunction<T:
U:
R>: 接收两个不同类型的输入，返回结果


关于Prompt
Zero-shot: 零样本提示，适合简单任务
One-Shot & Few-Shot Prompting： 一次性提示和少量提示，对于需要特定输出格式的任务尤其有用
System Prompting：系统提示，设定语言模型的整体背景和目标，定义模型应该做什么的”大局“，为模型的响应建立了行为框架、约束和高级目标，与具体的用户查询无关
Role Prompting：角色提示，指示模型采用特定的角色或人物，通过为模型分配特定的身份、专业知识或视角，可以影响其响应的风格、语气、深度和框架
Contextual Prompting：情境提示，通过传递情境参数为模型提供额外的背景信息，这种技术可以丰富模型对特定情境的理解，使其提供更相关、更个性化的响应，不会干扰主要指令
Step-Back Prompting：后退提示，通过首先获取背景知识，将复杂的请求分解为更简单的步骤，鼓励模型首先从当前问题”后退一步“，思考更广泛的背景、基本原理或与问题相关的一般知识，然后在处理具体问题
Chain of Thought (CoT)：思维链，鼓励模型逐步推理问题，提高复杂推理任务的准确性，生成中间推理步骤，类似于人类解决复杂问题的方式
Self-Consistency：自一致性，通过对同一问题采用不同的推理路径，挺通过多数表决选择最一直的答案来解决LLM输出的差异性，通过生成具有不同温度或采样设置的多条推理路径，然后汇总最终答案，自一致性可以提高复杂推理任务的准确性。它本质上是一种 LLM 输出的集成方法。
Tree of Thoughts (ToT)：思维树，同时探索多条推理路径来扩展思维链，它将问题解决视为一个探索过程，模型会生成不同的中间步骤，评估其可行性，并探索最具有潜力的路径
Automatic Prompt Engineering：自动提示语工程，利用语言模型本身来创建、改进和基准测试不同的提示语变体，找到针对特定任务的最佳方案
Code Prompting：代码提示，利用LLM理解和生成编程语言的能力，使其能够编写新代码、解释现有代码、调试问题以及在语言之间进行转换


Agent框架

┌──────────────────────────────┐
│           人类自然语言指令         │  ←（你说的话）
└──────────────────────────────┘
                  ↓
┌──────────────────────────────┐
│       大脑：理解与决策模块（LLM）   │  
│ - 理解意图（Intent Recognition） │
│ - 任务分解（Task Decomposition） │
│ - 规划步骤（Planning）            │
└──────────────────────────────┘
                  ↓
┌──────────────────────────────┐
│         动作管理器（Action Manager） │ 
│ - 选择合适的“手脚”来执行             │
│ - 组合多个动作（Action Composition）│
└──────────────────────────────┘
                  ↓
┌───────────────────────────────────────────────────────┐
│                       手脚执行层（Execution Layer）   │
│ ┌───────────────────────┬────────────────────────┐  │
│ │  计算机手脚（Computer Use） │   物理世界手脚（Robotics） │
│ │  - 控制应用/浏览器          │   - 机械臂控制           │
│ │  - 文件系统操作             │   - 融合拾取（视觉+触觉） │
│ │  - API调用自动化            │   - 运动规划             │
│ └───────────────────────┴────────────────────────┘  │
└───────────────────────────────────────────────────────┘
                  ↓
┌──────────────────────────────┐
│            反馈监控（Feedback Loop）  │
│ - 检查执行结果是否成功          │
│ - 自我修正（Self-Correction）   │
└──────────────────────────────┘


AI如何识别需要联网？
识别机制    说明
🧠 模型自身推理    基于常识、截止时间、问题意图
🛠️ 工具函数 schema  注册函数后模型会自动学会调用
🧩 函数调用流程    模型 → 指令 → 系统执行 → 结果返回模型
🤖 与用户交互时    用户看不到调用细节，只看到自然语言结果


对于开发者而言，函数调用的实现机制细节：
1. 函数注册 + 工具描述（Function Schema）
模型预先知道有哪些“函数”可以调用，以及参数说明。
2. 模型输出 function_call 或 tool_call 指令
当模型识别出需要联网时，会自动构造调用指令。
3. 系统层执行实际 API 请求
模型本身不能联网，但系统根据模型指令去联网。



1、LLM 如何实现基于 OpenAI 的对话系统？如何支持 Function Call？
[ 用户请求 ]
     ↓
[ Java 后端 ]
     ↓
[ 调用 OpenAI Chat API (支持 function_call) ]
     ↓
[ 检测是否触发函数调用 ]
     ↓
[ 执行本地函数 → 返回结果 ]
     ↓
[ 将结果回传给 OpenAI 继续对话 ]
     ↓
[ 返回最终响应给用户 ]

原 Function Call: functions 参数，调用字段是function_call
MCP 模式（tool_calls）：tools 参数（type="function"），调用字段是tool_calls（支持多个工具）

public class OpenAIMcpExecutor {
    public ChatCompletionResult execute(List<Message> messages, List<ToolFunction> tools) {
        // 1. 发起第一轮请求，获得 tool_calls
        // 2. 根据 tool_calls 执行本地函数，生成 tool 响应
        // 3. 拼接 tool 响应，继续调用 OpenAI
        // 4. 返回最终 assistant 回复
    }
}

--------------------------------------------------------------------------------------------------------
2、Flux/SSE    如何在 WebFlux 中返回流式响应？
方式一：返回 Flux<T>（默认返回 application/json 的流）支持JSON、NDJSON
public Flux<Integer> streamNumbers() {
    return Flux.range(1, 10).delayElements(Duration.ofSeconds(1));
}
方式二：返回 Flux<ServerSentEvent<T>>（SSE）
public Flux<ServerSentEvent<String>> streamMessages() {
    return Flux.interval(Duration.ofSeconds(1))
            .map(seq -> ServerSentEvent.<String>builder()
                    .id(String.valueOf(seq))
                    .event("message")
                    .data("Message " + seq)
                    .build()
            );
}
| 项目           | `Flux<T>`              | `Flux<ServerSentEvent<T>>`  |
| ------------ | ---------------------- | --------------------------- |
| 适用格式         | JSON、NDJSON            | 文本、结构化 SSE                  |
| Content-Type | `application/ndjson`   | `text/event-stream`         |
| 支持事件/ID      | 否                      | ✅ 支持 id / event / retry 等字段 |
| 客户端兼容性（浏览器）  | 差（需特殊处理）               | ✅ 原生支持                      |
| 前端接收方式       | fetch + ReadableStream | `EventSource`               |

NDJSON（Newline Delimited JSON，也称为 JSON Lines）是一种用于处理大量结构化数据的轻量级数据格式。它的核心思想是：每一行是一条独立的 JSON 对象（标准 JSON），多个对象之间以换行符 \n 分隔。

--------------------------------------------------------------------------------------------------------
3、架构  如果让你设计一个多租户 AI 应用系统，你会如何划分模块？
┌────────────────────────────┐
│        API Gateway         │
└────────────┬───────────────┘
             │
┌────────────▼────────────┐
│    Authentication/Authz │  ← 多租户身份认证与权限控制
└────────────┬────────────┘
             ▼
 ┌────────────────────────┐
 │   Tenant Management     │ ← 租户信息、资源配额、隔离策略
 └────────────────────────┘
             │
┌────────────▼────────────┐
│     Application Layer    │
├────────────┬────────────┤
│ Chat Engine│ Embedding   │ ← AI 能力层（可扩展多模型）
│ FunctionCall│ Search/RAG │
└────────────┴────────────┘
             │
┌────────────▼────────────┐
│     Vector Store Layer   │ ← 租户级嵌入、知识库等
└────────────┬────────────┘
             ▼
┌──────────────────────────┐
│         Storage           │ ← 用户数据、模型调用记录等
└──────────────────────────┘
AI 服务能力模块：
| 模块                 | 功能                    | 特点                    |
| ------------------ | --------------------- | --------------------- |
| `ChatEngine`       | 支持 OpenAI / Gemini 对话 | 支持 Function Call / 插件 |
| `EmbeddingService` | 文本转向量、向量存储            | 用于 RAG                |
| `FunctionCall`     | 接入业务函数 / 工具调用         | 多租户可自定义函数             |
| `RAGEngine`        | 基于嵌入+检索生成回答           | 对接向量数据库               |
| `PromptTemplate`   | 租户可配置 prompt 模板       | 支持 prompt 工程能力        |


function call和RAG对比：
| 维度                             | Function Call                  | RAG (Retrieval-Augmented Generation) |
| ------------------------------ | ------------------------------ | ------------------------------------ |
| ✅ **核心作用** | 执行函数，调用外部 API、工具等              | 检索外部知识，增强模型知识                        |
| ✅ **适合场景** | 查询数据库、天气、调用计算器、插件、执行代码等        | 问答系统、文档问答、知识库接入等                     |
| ✅ **触发方式** | 模型生成函数名 + 参数，平台自动调用后返回结果       | 模型先基于提问去向知识库检索，再连同上下文喂给模型            |
| ✅ **需要结构** | 需要提前定义函数规格（如 FunctionSpec）     | 需要检索系统（如向量库、全文搜索）                    |
| ✅ **是否依赖外部知识库**| ❌ 不依赖知识库，但依赖接口（如函数/工具）         | ✅ 依赖向量数据库、文档等                        |
| ✅ **是否动态调用**| ✅ 模型可以按需调用函数或多轮协商              | ✅ 检索系统动态选择最相关上下文                     |
| ✅ **OpenAI 支持** | 支持 function_call / tool_call | 支持，通过提供内容作为上下文                       |
| ✅ **Spring AI / LangChain 支持** | ✅ 明确支持（Function tool）          | ✅ 明确支持（retriever + chain）            |

如果模型需要调用多个tool：
{
  "role": "assistant
  "tool_calls": [
    {
      "id": "call_1
      "function": {
        "name": "getWeather
        "arguments": "{ \"location\": \"Beijing\" }"
      }
    },
    {
      "id": "call_2
      "function": {
        "name": "getAirQuality
        "arguments": "{ \"city\": \"Beijing\" }"
      }
    }
  ]
}

--------------------------------------------------------------------------------------------------------
4、性能  如何优化高并发下的 AI 请求系统？如何避免模型接口被滥用？
一、系统架构优化：提升并发处理能力
1. 使用响应式框架（如 Spring WebFlux）非阻塞IO，适合AI请求
2. 引入消息队列/异步任务 将 AI 请求变为异步任务，提高用户响应速度。
3. 弹性扩展（Auto Scaling） 部署在 Kubernetes / 云平台（如 GKE、ECS）中，根据负载自动水平扩展服务实例；AI 模型调用可以通过任务队列/负载均衡分摊压力。
二、限流、防刷机制：防止接口滥用
1. API Key 管理 用户独立分配key，请求必须携带key，记录key的使用次数、调用日志
2. 限流策略（Per User/Tenant/IP）使用 Bucket4j、resilience4j、RateLimiter 实现令牌桶或漏桶；支持按用户/租户维度限流
3. 验证码 / 人机验证
4. 模型调用计费和配额
三、缓存与复用策略
1. Prompt+Response 缓存 相同请求体可命中缓存，避免重复调用 AI 接口。可基于 Hash(prompt+params) 作为缓存 Key。
2. Embedding 缓存 相同文本的 embedding 向量可缓存。使用 Redis / 本地缓存（如 caffeine）+ 向量存储。
3. 对话上下文裁剪 对话历史越长，请求越慢。可以：滑动窗口保留最近 n 条消息。向量召回精要历史
四、安全与隔离机制
1. 租户隔离
2. 模型代理/中转服务 禁止客户端直接请求 OpenAI / Gemini API，强制经由你的服务转发。防止泄露 API Key 并支持统一限流和日志追踪。
3. 加签请求 & 防重放 所有客户端请求加签名（HMAC），并加时间戳。避免请求伪造或重放攻击。
五、AI调用性能优化
1. 模型选择与多模型调度 使用不同模型处理不同级别请求。可接入多个服务商（OpenAI、Gemini、Claude）动态分流。
2. 流式输出 vs 批量处理 对于批量任务，聚合处理（如 embedding 批量请求）。
3. 模型延迟监控与熔断 监控请求响应时间，超时或异常时降级处理。引入熔断器（如 Resilience4j）避免雪崩效应。
六、可观测性与运维 监控：Prometheus + Grafana 监控调用量、QPS、延迟。日志追踪：ELK / Loki + tenant_id / user_id 打日志。审计日志：记录所有模型调用历史、参数、返回值。

--------------------------------------------------------------------------------------------------------
5、数据  怎么存储用户对话历史？如何裁剪历史来控制 Token？
一、如何存储用户对话历史
1. ✅ 存数据库（推荐） 可持久化、查询方便。支持多轮历史回溯、分页。支持多租户隔离（通过 user_id / tenant_id）
2. 存缓存（如 Redis）适合短期记忆、性能优先的场景。
二、如何裁剪历史以控制 Token 数量
OpenAI 等模型有 Token 上限，比如 GPT-4 是 128k、GPT-3.5 是 16k（或 4k）。为了不超出限制、提升性能，要进行 裁剪（truncation）。
✅ 1. 滚动窗口（Sliding Window） 只保留最近 N 条对话，如最近 10 条消息。实现简单。
✅ 2. Token 计数裁剪（推荐） 每次请求前计算当前总 Token 数量，超过设定上限时，从最旧的对话开始裁剪。
实现：使用 OpenAI 提供的 tiktoken 或开源 GPTTokenizer 工具进行 Token 计算。不断累加历史消息的 token，直到接近最大值时停止加入。
✅ 3. 优先保留关键信息 system prompt 始终保留。用户自定义上下文保留（如知识库）。对长回答进行摘要压缩（可用 GPT 总结历史）。
| 项目         | 建议/方案                       |
| ---------- | --------------------------- |
| 存储方式       | 数据库（长期）+ Redis（短期）          |
| 裁剪策略       | 滚动窗口 or Token 计数            |
| Token 控制方式 | 估算 Token 数，裁剪旧消息            |
| 保留内容优先级    | system prompt > 最近上下文 > 旧对话 |
| 高级优化       | 总结压缩、向量搜索、长对话记忆模块           |

--------------------------------------------------------------------------------------------------------
6、DevOps  你是如何部署你的 AI 应用的？是否使用容器？如何监控？
[Client UI]
    ↓ REST/SSE
[Java 后端 API 服务] ——→ [OpenAI/Gemini 接口调用]
    ↓ Redis 缓存       ↘︎
    ↓ Vector Store (如 Weaviate / Milvus)
    ↓ MySQL 存储用户对话历史
  ↘︎ Prometheus + Grafana 监控

| 部署组成      | 技术选型                                |
| --------- | ----------------------------------- |
| 应用运行环境    | Docker / Kubernetes                 |
| 配置管理      | K8s ConfigMap / Spring Cloud Config |
| 服务发现 & 网关 | Nginx / K8s Ingress / API Gateway   |
| 缓存        | Redis                               |
| 数据库存储     | MySQL / PostgreSQL                  |
| 对话历史/向量检索 | MySQL + 向量库（如 Milvus）               |
| 日志/监控     | ELK / Loki / Prometheus + Grafana   |
| 安全 & 限流   | JWT、限流中间件、API Key 校验                |



在流式返回中 tool_calls 的行为：
1. 首次返回会是一个含有 tool_calls 的 delta 块，形式类似：这只是函数名的开始，不含参数。
{
  "choices": [
    {
      "delta": {
        "tool_calls": [
          {
            "id": "call_1
            "type": "function
            "function": {
              "name": "getWeather"
            }
          }
        ]
      }
    }
  ]
}
2. 接下来会返回多个包含 arguments 分片的 delta 块：
{
  "choices": [
    {
      "delta": {
        "tool_calls": [
          {
            "index": 0,
            "function": {
              "arguments": "{ \"location\": \"Be"
            }
          }
        ]
      }
    }
  ]
}
将这些片段拼接起来才能获得完整的参数字符串。
3. 如果有多个 tool_call，会在同一个流中逐步构建多个调用块，通过 index 区分：
"tool_calls": [
  { "index": 0, "function": { "name": ..., "arguments": ... } },
  { "index": 1, "function": { "name": ..., "arguments": ... } }
]
最终在 finish_reason == "tool_calls" 时构造出完整调用。


在流式返回中 FunctionCall 的表现：
name 和 arguments 都可能分块。累积拼接 arguments，在 finish_reason == function_call 时统一处理。FunctionCall 只支持一个函数，tool_calls 支持多个并通过 index 区分。


Flux和FluxSink的区别？
| 对比点   | `Flux<T>`                             | `FluxSink<T>`                       |
| ----- | -------------------------------- | ----------------------------------- |
| 是什么？  | 响应式数据流                            | 数据流的推送入口                            |
| 谁用它？  | 订阅者（消费者）                         | 生产者（开发者手动推送）                        |
| 如何创建？ | `Flux.just()`, `Flux.interval()` | `Flux.create(emitter -> {...})` 中获得 |
| 能做什么？ | 被动等待数据，订阅处理                | 主动 `.next()` 推送数据                   |
| 用途    | 消费数据                             | 产生数据（尤其是非响应式来源）                     |


| 类型                                     | 多个订阅者行为                 | 示例                       |
| -------------------------------------- | ----------------------- | ------------------------ |
| ❗ 普通 `Flux` / `Flowable`               | 每个订阅者 **独立消费**（重新执行流逻辑） | `Flux.fromIterable(...)` |
| ✅ `ConnectableFlux` / `PublishSubject` | 多个订阅者 **共享同一个数据源**（广播）  | 热流（Hot stream）或共享流       |
| ✅ `ReplaySubject`, `BehaviorSubject`   | 新订阅者也能收到历史数据            | 多订阅者共享数据和回放历史            |

| 问题                        | 答案                                        |
| ------------------------- | ----------------------------------------- |
| **两个订阅者，前者修改数据，后者会受影响吗？** | **可能会，取决于是否是共享的可变对象引用。**                  |
| **如何避免？**                          | 使用不可变对象、clone、`Flux.defer()` 或在每次订阅时新建对象。 |


介绍SharedInterpreter
1、什么是SharedInterpreter？
SharedInterpreter 通常来自 Jep (Java Embedded Python)，这是一个将 CPython 嵌入到 Java 应用程序中的库。它允许 Java 和 Python 高效地进行交互。
特点：①基于 CPython：不是 Jython，而是直接用原生 Python 实现。 ②可以在 Java 中调用 Python 函数、导入模块、传递数据。 ③支持线程隔离，每个线程用自己的解释器上下文。 ④性能高，适合在 Java 中运行小段 Python 脚本，特别是 AI/数据处理任务。
2、SharedInterpreter 的使用流程
①创建解释器对象，interp = new SharedInterpreter(); 这会初始化 Python 环境，等同于在 Java 中打开一个嵌入式 Python 会话。
②执行 Python 脚本，interp.exec(prefix + code); 该行会把字符串形式的 Python 脚本传入解释器执行。exec() 会定义函数、变量，但不会执行函数调用。
③调用函数，codeRes = interp.invoke("main", kwargs); 这一步是核心：在 Python 中调用名为 main() 的函数，并将 Java Map（kwargs）自动转换为 Python 的参数。 
④返回结果给 Java，函数 main() 的返回值通常是 str（如 JSON 字符串），Jep 会将其自动转换为 Java 的 Object（一般是 String 或 Map）。
3、SharedInterpreter 典型使用场景
| 场景      | 描述                                 |
| ------- | ---------------------------------- |
| AI 推理   | 在 Java 中调用 PyTorch、Transformers 模型 |
| 数据处理    | 用 Pandas/Numpy 快速处理数据              |
| 自定义函数执行 | 用户上传或编辑 Python 脚本，动态执行             |
| 可视化     | 用 Python 生成图像或动画结果，返回到 Java UI     |
4、内存和线程注意事项
①资源释放：interp.close(); 每次使用后必须释放资源，否则会导致内存泄漏或文件句柄耗尽。
②线程隔离：SharedInterpreter 是线程安全的，但不同线程必须使用不同实例。
③GIL（全局解释器锁）：因为使用的是 CPython，本质上仍受 GIL 影响，不适合并发密集调用。
用 Java + Python（如用 Jep），GIL 也会起作用，但每个 SharedInterpreter 实例默认是线程隔离的，不共享 GIL，不会出现线程争抢问题。


Easy Rules 与其他规则引擎对比：
| 特性 / 引擎    | **Easy Rules**            | **Drools**                 | **RuleBook**        | **JBoss Rules (旧名)** |
| ---------- | ------------------------- | -------------------------- | ------------------- | -------------------- |
| **定位**     | 轻量级规则引擎               | 功能强大、企业级规则引擎        | 注重函数式、链式规则表达   | Drools 的前身           |
| **规则编写方式** | Java DSL / 注解 / JSON/YAML | DRL (Drools Rule Language) | Java 8 Lambda / 函数式 | DRL                  |
| **学习曲线**   | 非常低                       | 较高，需要学习 DRL 语法        | 中等，需要理解函数式编程  | 同 Drools             |
| **引擎复杂度**  | 简单、直观                     | 强大但复杂                   | 中等              | Drools 分支            |
| **性能**     | 适合中小型规则场景                 | 高性能，适用于复杂场景         | 中等                | -                    |
| **规则优先级**  | 支持 @Priority              | 支持 salience                | 支持条件链            | 支持                   |
| **规则间依赖**  | 不支持复杂依赖                 | 支持规则推理和链式执行          | 支持前置规则链         | 支持                   |
| **使用场景**   | 表达简单业务逻辑，微服务中        | 复杂业务规则、工作流系统         | 基于 Java8 的业务流控制 | 同 Drools             |
| **文档与社区**  | 一般                        | 丰富的文档与社区支持            | 社区相对较小           | 逐渐被淘汰                |

Easy Rules 优点
💡 上手简单：使用 Java 语法即可定义规则，零学习成本。
💼 易于集成：非常适合 Spring Boot 等轻量框架。
📄 支持 YAML/JSON 规则配置，便于从配置中心或数据库动态加载规则。
🧱 支持规则组合（规则组、组合规则），提升复用性。
🛠️ 支持优先级、条件判断、动作执行 等常用功能。
Easy Rules 限制
①不支持复杂推理（如基于事实的推理）。
②不支持规则间的数据流依赖和冲突解决（如 conflict resolution）。
③适用于“规则执行是否满足”的简单场景，不适合专家系统或复杂决策引擎。

适用场景对比总结
| 场景                   | 推荐引擎                  |
| -------------------- | --------------------- |
| 简单决策逻辑、流程控制（if/else） | ✅ Easy Rules          |
| 复杂业务规则、多条件组合         | ✅ Drools              |
| Java8 函数式规则构建        | ✅ RuleBook            |
| 专家系统、规则依赖/冲突         | ✅ Drools              |
| 配置化规则从数据库加载          | ✅ Easy Rules / Drools |



OpenManus
https://blog.csdn.net/qq_35082030/article/details/146371952


向量数据库
https://www.pinecone.io/learn/vector-database/
向量数据库的三个通用流程：indexing、querying、post processing
1、创建索引的常用算法：
①Random Projection（随机投影）
②Product Quantization（PQ）
③Locality-sensitive hashing（LSH）
④Hierarchical Navigable Small World (HNSW)
2、相似度计算：Cosine Similarity、Euclidean distance、Dot product
3、过滤：post-filtering、pre-filtering

可以重点关注PineCone（项目中用到）、Milvus（开源）
| 特性 / 产品         | Pinecone | FAISS | Milvus | Qdrant | Weaviate |
| --------------- | -------- | ----- | ------ | ------ | -------- |
| 部署模式            | 托管       | 本地库   | 云原生    | 可自建    | 可自建      |
| 分布式             | ✅        | ❌     | ✅      | ✅      | ✅        |
| Metadata filter | ✅        | ❌     | ✅      | ✅      | ✅        |
| REST API        | ✅        | ❌     | ✅      | ✅      | ✅        |
| SDK 易用性         | 极佳       | 中等    | 中等     | 较佳     | 中等       |
| 性能优化            | 自动扩缩容    | 手动调参  | 手动部署   | 高效轻量   | 模块化丰富    |
| 商业化支持           | ✅（闭源）    | ❌     | ✅      | ✅      | ✅        |

以向量为操作对象
向量数据库结合使用多种参与近似最近邻 (ANN) 搜索的算法。这些算法通过哈希、量化或基于图的搜索来优化搜索。
索引：向量数据库使用 PQ、LSH 或 HNSW 等算法对向量进行索引（详见下文）。此步骤将向量映射到一个数据结构，以便更快地进行搜索。
查询：向量数据库将索引查询向量与数据集中的索引向量进行比较，以查找最近邻（应用该索引使用的相似度度量）。
后处理：在某些情况下，向量数据库会从数据集中检索最终的最近邻，并对其进行后处理以返回最终结果。此步骤可能包括使用不同的相似度度量对最近邻重新排序。

一、K-Means + Voronoi 分区
步骤如下：
1、预聚类训练阶段（离线）
使用 K-Means 算法将向量训练集聚类为 K 个中心向量（centroids）：每个中心代表一个“分区”
2、构建 Voronoi 区域
对于任意查询向量，计算其与每个中心的距离，将它归入最近的中心区域，这些区域的划分就叫做 Voronoi 区域。
3、每个区域内建立子索引
将属于每个聚类中心的向量归为一类，为其建立子索引：使用 HNSW/PQ/Flat 等独立索引结构
4、存储在不同节点或存储区
查询时仅查询 Top-N 个区域
通过 coarse quantizer（粗排器），先查找最相关的几个区域，然后在它们的子索引中进行精确搜索。

二、Product Quantization (PQ) 中“压缩 + 码本查表”的完整实现原理
一句话总结：PQ 把高维向量压缩成低位表示（如 64-bit 编码），通过 子空间划分 + 码本查表 来加速近似相似度计算。
PQ 的过程可以拆解为两个阶段：
1、离线建码本（训练阶段）
①将每个 D 维向量切成 M 个子向量段，每段维度为 D / M
②对每段子向量分别做 k-means 聚类（k 通常为 256）
③记录每个子空间的 k 个聚类中心（即码本）
这样就构造了一个大小为 M × k × (D/M) 的 3D 码本表。
2、在线压缩（编码阶段）
给一个向量 x ∈ ℝ^D，我们把 x 切成 M 段子向量：x₁, x₂, ..., xₘ
对每段 xᵢ，找到该段码本中最接近的中心 cᵢⱼ。记录这个中心的索引 j，作为压缩编码的一部分。
最终，每个向量 x 被表示成一个长度为 M 的 编码向量 [j₁, j₂, ..., jₘ]，即每个子向量的“最近中心”编号。
③ 在线查表（计算相似度）
查询向量 q 也按相同方式切分为子向量 q₁, q₂, ..., qₘ，然后进行：对于每段 i，先和码本中心 cᵢⱼ 计算距离（提前算好）。
对于向量 x，其编码 [j₁, ..., jₘ]，对应的距离为：d(q, x) ≈ Σᵢ || qᵢ - cᵢⱼᵢ ||²

PQ 的数据结构与计算：
码本（Codebook）：二维数组 float[M][K][D/M]
编码（Code）：向量编码为 byte[M] 数组
查表距离：每次查询前构建距离表 float[M][K]，然后加和得到距离

三、HNSW（Hierarchical Navigable Small World）
一句话总结：HNSW 是一种图索引算法，通过建立分层小世界图结构，使得向量搜索在大规模数据中也能快速找到近似最邻近结果。
1、核心思想：分层近邻图（HNSW 的核心是构造一个分层的图（近邻网络），使得搜索可以从“高层粗粒度导航”到“底层精细搜索”，快速定位向量。）
图的结构：①向量构成图中的 节点。②相似的向量之间连边（即邻居）。③节点按概率被分配到多个“图层”，形成一个多层结构。
每一层是一个小世界图（Navigable Small World），高层稀疏、低层密集。
2、构建算法（离线构图）
设一个向量 v 要插入图中，步骤如下：
①确定插入的最高层 L
a.用概率 P(level = l) = e^{-λl}，比如 λ=1，L 满足概率下降的幂律分布。
b.高层节点越少，整体层数是对数级别。
②从最高层往下插入
对于每一层，从高层开始，逐层往下：
a.使用贪心搜索：从某个入口节点开始，逐步跳到更近的邻居，直到找不到更近的。
b.找到目标点附近的若干个近邻，连边。
c.若边数超过 M，用 heuristics（如多样性）选最优。
③维护邻居数量限制
a.每个节点在每层的邻居数都受到限制（如 M=16），可提升查询效率。
3、查询算法（在线搜索）
给一个查询向量 q，要找其近邻：
①从最顶层 Entry Point 开始
a.用贪心算法，逐步跳向更接近的点。
b.每次从当前点的邻居中选择更近的向量。
②向下逐层导航
a.到达底层时，执行更精细的 局部搜索（多候选 BFS）。
b.使用优先队列（Max-Heap）维护 top-k 最近邻。
③返回 k 个最近邻结果
4、数据结构
多层图结构 layers: List<GraphLayer>
每层是一个图 GraphLayer: Map<NodeID, List<NeighborNode>>
每个节点存有 level、vector、邻居列表

分层原理：在HNSW索引结构中，每个节点所在的层数是通过指数概率分布随机决定的，不是人为指定的。这种层级的设计，借鉴了SkipList的思想，使得查询可以“从高层粗导航到低层精搜索”。对于层数 l，出现的概率为：P(level=l)=e^(−λl)（一般默认 λ = 1 / \ln(M)，其中 M 是每层最大连接数。）

结对比 PQ vs HNSW
| 特性   | PQ             | HNSW            |
| ---- | -------------- | --------------- |
| 类型   | 压缩编码 + 查表      | 图结构搜索           |
| 优点   | 占用空间小，批量比对快    | 搜索精度高，动态可扩展     |
| 查询速度 | 快（近似）          | 更快（高精度）         |
| 支持插入 | 一般较慢（静态结构）     | 快速支持新增向量        |
| 应用   | IVF-PQ / PQ子索引 | HNSW原生或与IVF组合使用 |


何为向量数据库中的元数据？——在向量数据库中，元数据（Metadata）是与向量一起存储的结构化信息或描述性信息，用于标注、筛选、过滤、排序或查询向量内容。
无服务架构的向量数据库是什么样的？——是指用户无需管理服务器资源（如 CPU、内存、分片、副本等），由云服务商动态调度计算与存储资源、按需计费的向量数据库系统。
怎么做到索引存储与查询分离？—— 通过划分向量空间几何结构 来实现**索引分片（sharding）或子索引构建（sub-indexing）**。它的目标是将一个大规模的向量集合分成多个更小、更独立的区域，每个区域维护一个子索引，从而实现 存储与计算分离、并行加速 和 可伸缩性增强。


涉及模型的主要发展方向：
模型性能、模型优化能力、基础设施、数据库、开发平台、运维、部署、安全及开发社区等9大维度


Java MCP

+-----------------------------------------------------------------------------------------------------+
|                                      io.modelcontextprotocol                                         |
+-----------------------------------------------------------------------------------------------------+
                                                |
                +-----------------------------+--+---------------------------+------------------------+
                |                             |                            |                        |
        +-------v-------+             +-------v-------+            +-------v-------+        +-------v-------+
        |    client     |             |    server     |            |     spec      |        |     util      |
        +---------------+             +---------------+            +---------------+        +---------------+
                |                             |                            |                        |
    +-----------+-----------+     +-----------+-----------+    +-----------+-----------+  +-----------+----------+
    |           |           |     |           |           |    |           |           |  |           |          |
+---v---+ +-----v----+ +---v---+ +---v---+ +---v---+ +---v---+ +---v---+ +---v---+ +---v---+ +---v---+ +---v---+
|McpCli-| |McpAsync-| |McpSync| |McpSer-| |McpAsync| |McpSync| |McpSch-| |McpErr-| |McpTra-| |Utils  | |Assert |
|ent    | |Client   | |Client | |ver    | |Server | |Server | |ema    | |or     | |nsport | |       | |       |
+-------+ +----------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+ +-------+
    |           |           |        |        |         |          |         |         |         |         |
    |           |           |        |        |         |     +----v----+    |    +----v----+    |    +---v---+
    |      +----v----+     |   +----v----+   |         |     |JSONRPCMe|    |    |McpClient|    |    |McpUri-|
    |      |McpAsync|      |   |McpServer|   |         |     |ssage    |    |    |Transport|    |    |Templa-|
    |      |ServerEx|      |   |Features|   |         |     +---------+    |    +---------+    |    |teMana-|
    |      |change  |      |   +---------+   |         |         |         |         |         |    |ger    |
    |      +---------+     |        |        |         |    +----+----+    |    +----+----+    |    +-------+
    |           |          |        |        |         |    |JSONRPCRe|    |    |McpServer|    |         |
    |           |          |        |        |         |    |quest    |    |    |Transport|    |    +----v----+
    |      +----v----+     |        |     +---v---+   |    +---------+    |    +---------+    |    |McpUriTe-|
    |      |McpSync |      |        |     |McpSync|   |         |         |         |         |    |mplateMa-|
    |      |ServerEx|      |        |     |ServerE|   |    +----v----+    |         |         |    |nagerFac-|
    |      |change  |      |        |     |xchange|   |    |JSONRPCRe|    |    +----v----+    |    |tory    |
    |      +---------+     |        |     +-------+   |    |sponse   |    |    |McpServer|    |    +---------+
    |                      |        |                 |    +---------+    |    |Transport|    |         |
+---v----------------------v--------v-----------------v----+             |    |Provider |    |    +----v----+
|                      transport                          |             |    +---------+    |    |Deafault-|
+----------------------------------------------------------+             |         |         |    |McpUriTe-|
    |                      |                      |                      |         |         |    |mplateMa-|
+---v----+          +------v------+        +------v------+        +-----v-----+   |         |    |nagerFac-|
|FlowSse-|          |HttpClientSs|        |StdioClient  |        |HttpServlet|   |         |    |tory    |
|Client  |          |eClientTran-|        |Transport    |        |SseServerT-|   |         |    +---------+
+--------+          |sport       |        +-------------+        |ransportPr|   |         |
    |               +-------------+                              |ovider    |   |         |
+---v----+                                                       +-----------+   |         |
|ServerPa|                                                                       |         |
|rameters|                                                                       |         |
+--------+                                                                       |         |
                                                                          +-----v-----+   |
                                                                          |McpSession|   |
                                                                          +-----------+   |
                                                                               |         |
                                                                        +------+------+  |
                                                                        |             |  |
                                                                  +-----v------+ +----v---+
                                                                  |McpClientSe-| |McpServ-|
                                                                  |ssion       | |erSessio|
                                                                  +------------+ +---------+
                                                                        |            |
                                                                  +-----v------+     |
                                                                  |McpTranspor-|     |
                                                                  |tSession    |     |
                                                                  +------------+     |
                                                                        |            |
                                                                  +-----v------+     |
                                                                  |DefaultMcpT-|     |
                                                                  |ransportSes-|     |
                                                                  |sion        |     |
                                                                  +------------+     |


---------------------------------------------------------------------------------------------------------------------------------------------------------
AutoCloseable总结：
1、AutoCloseable 让你的类支持 try-with-resources 自动关闭；
2、避免忘记手动关闭资源，防止资源泄漏；
3、可应用于 IO、数据库、自定义连接等需要释放的资源；
4、实现后，只需实现 close() 方法即可。

ContextView：
ContextView 提供了只读方式访问 Reactor 中流的上下文数据（类似于线程局部变量 ThreadLocal，但适用于响应式编程环境）。
它提供了流式调用过程中与线程无关的上下文访问能力，常用于构建与请求强相关的对象。类似于线程局部变量 ThreadLocal。

private static final Pattern EVENT_DATA_PATTERN = Pattern.compile("^data:(.+)$", Pattern.MULTILINE);匹配多行以data:开头的内容。





以下是一个【更全面的 AI 应用弹性架构方案】，核心目的是：解决从用户请求到模型推理的全链路痛点，兼顾「高并发削峰」「可控时延」「稳定算力调度」「推理回退容错」等多维度能力。

---

## ✅ 一、AI 应用常见痛点回顾

| 典型痛点        | 场景表现                       |
| ----------- | -------------------------- |
| ❶ 短时间请求激增   | 高峰期用户爆发、热点话题、大促流量等         |
| ❷ 长短推理混杂    | 普通对话和复杂推理共存，容易资源被长推理拖死     |
| ❸ 异常推理阻塞    | 单条推理过慢或失败导致整体雪崩            |
| ❹ 会话上下文丢失   | SSE / WebSocket 短连断开导致消息丢失 |
| ❺ GPU算力弹性不足 | 静态部署，AI服务无法按需弹性伸缩          |

---

## ✅ 二、推荐弹性架构方案（全景图解构）

### 1. **前端层**

* ✅ 用户通过 HTTP/WS/SSE 接入
* ✅ 引入请求特征（VIP、内部、公共流量等）

### 2. **流控调度层（MQ）**

* ✅ RocketMQ/Kafka 削峰填谷
* ✅ 会话/用户维度 Topic 或分区：保障单会话有序
* ✅ 优先级 Topic / 延时队列：保障重要用户低时延

### 3. **推理调度层**

* ✅ 推理请求调度器（自研或 K8S Operator）按优先级拉取消息
* ✅ 结合 Fast Slot (小模型)，Full Slot（大模型）
* ✅ 并行推理、超时快断

### 4. **推理服务层**

* ✅ GPU/CPU 混合推理
* ✅ 推理分级：

  * FastModel（轻量模型）快速响应
  * FullModel（重模型）高质量推理
* ✅ 支持 fallback fallback：

  * FastModel fallback
  * API fallback（如搜索召回）
  * 缓存 fallback（如相似回答）

### 5. **弹性算力层**

* ✅ K8S 动态伸缩（HPA + VPA）
* ✅ 算力负载均衡（推理服务通过 sidecar 分发请求）
* ✅ Spot GPU / On-demand GPU 弹性调度

### 6. **持久化与监控**

* ✅ 会话上下文缓存（Redis / KV DB）
* ✅ MQ 日志追踪链路完整性
* ✅ Prometheus + Grafana 全链路指标监控
* ✅ 失败请求重试（MQ 重入队列 + 异步补偿）

---

## ✅ 三、关键“弹性”机制总结

| 维度      | 弹性设计                |
| ------- | ------------------- |
| ⏳ 请求时延  | MQ削峰+优先级调度          |
| 📈 算力扩缩 | GPU容器弹性 + spot 节省成本 |
| 💥 推理容错 | 超时断流+fallback 回退方案  |
| 📡 断连恢复 | SSE重连+会话上下文恢复       |
| 💡 资源利用 | 短推理快路+长推理缓路+延迟补偿    |

---

## ✅ 四、整体架构参考图（可选）

📝 如果你需要，我可以直接为你生成一张【AI 应用弹性架构图】（可配合 RocketMQ 方案）：

```
[Client] → [流控(MQ)] → [推理调度] → [推理服务(GPU/CPU)] → [回退策略] → [监控+补偿]
```

---


万智2.0
一，能基于企业场景的交付目标来规划任务、跨部门跨组织多工具串联，形成交付结果的闭环；
二，能校验+评估保证质量，降低幻觉、保障结果；
三，能不断消化公域与私域知识，基于任务反馈自我迭代；
四，跨越手机和PC，实现互通终端互联系统；
五，支持接入和调用企业现有工作流，实现两天或者甚至更快的时间快速上岗使用。




《Introduction to Machine Learning》
这篇论文是由Laurent Younes撰写的，题为《Introduction to Machine Learning》，旨在提供机器学习的全面介绍，涵盖其数学基础、统计方法和算法实现。论文内容十分详尽，结构合理，分为多个章节，每个章节都涵盖了机器学习中的重要主题和技术细节。

核心方法论
本书的核心方法论主要集中在以下几个方面：

数学基础：

书中首先介绍了线性代数、拓扑学、微积分和概率论的基本概念。这些内容为理解后续的机器学习算法打下了坚实的基础。书中特别强调了线性代数的应用，例如矩阵运算、特征值分解和奇异值分解（SVD），这些都是后续数据分析和机器学习的重要工具。
矩阵分析：

论文的第二章深入探讨了矩阵理论的一些重要结果，阐述了矩阵运算的基本性质及其在机器学习中的应用。例如，书中介绍了Von Neumann的迹不等式和奇异值分解，这些是在处理优化问题和学习算法时经常用到的工具。
优化：

优化在机器学习中起着核心作用。书中对无约束与有约束优化问题进行了详细分析，包括最优性条件、梯度下降法、随机梯度下降（SGD）和Lagrange乘子法等。特别是，随机梯度下降法被详细讨论，它是现代机器学习模型训练中的基础算法之一。
统计推断：

论文对于统计学习的基本概念进行了介绍，包括参数估计、偏差-方差权衡和模型评估。书中提到的经验风险最小化原则为后续的模型训练提供了理论基础。
生成模型与判别模型：

书中对监督学习和无监督学习进行了深入讨论，包括线性回归、逻辑回归、支持向量机（SVM）、决策树以及集成学习方法（如随机森林和Boosting）。这些是机器学习中的常见方法，书中分别对它们的数学原理和实现策略进行了详细的分析。
神经网络与深度学习：

在谈到神经网络时，论文不仅介绍了基本的前馈神经网络，还探讨了更复杂的架构，包括卷积神经网络（CNN）和递归神经网络（RNN）。此外，书中也介绍了深度学习中的一些先进方法，如生成对抗网络（GAN）和变分自编码器（VAE）。
聚类和降维：

书中还包括了聚类分析的方法和技巧，例如K均值、层次聚类和谱聚类等。同时，降维技术如主成分分析（PCA）和流形学习等也得到了详细讨论，这些方法在高维数据处理中极为重要。
方法论的技术细节
在具体的技术细节方面，作者对每种算法的数学原理进行了推导，例如：

随机梯度下降：介绍了其收敛性分析和学习率选择的影响，同时给出了其在实际应用中的变种，例如Adam优化算法，强调了动态调整学习率的重要性。

模型评估：通过引入交叉验证和偏差-方差分解的方法，帮助读者理解模型的泛化能力和选择合适模型的重要性。

生成对抗网络：详细阐述了GAN的架构设计，包括生成器和判别器的交互，以及损失函数的选择及其优化技巧，使得读者能够切实掌握该先进技术的应用。

总的来说，这篇论文通过系统性且详细的方式介绍了机器学习领域的数学基础和技术细节，使得读者能够在理论和实践中都能够运用机器学习的工具和方法。




关于回调机制
核心思想是：将一段可执行的代码传递给另一个对象，由这个对象在合适的时机执行它。这种模式是实现“控制反转”的典型方式，广泛用于异步编程、事件驱动、框架扩展点等场景。
组成：调用方、被调用方、回调函数
分类：
| 类型    | 描述                      | 示例                              |
| ----- | ----------------------- | ------------------------------- |
| 同步回调  | 被调用方在执行过程中立刻调用回调函数      | Comparator 排序                   |
| 异步回调  | 被调用方在任务完成后，**异步触发回调函数** | 异步 HTTP、数据库、线程池任务               |
| 接口式回调 | 通过定义接口 + 实现回调           | Spring `RowMapper`              |
| 函数式回调 | 通过函数、Lambda 表达式         | Java 8 `Consumer<T>`、`Runnable` |
Java中回调的实现方式：
1）使用接口回调（经典写法）
public interface Callback {
    void onSuccess(String result);
}
public class Worker {
    public void doWork(Callback callback) {
        String result = "Done";
        callback.onSuccess(result);
    }
}
// 使用
worker.doWork(result -> System.out.println("回调结果：" + result));
2）使用 Java 8 Lambda 实现回调
public void runAsync(Runnable callback) {
    new Thread(callback).start();
}
runAsync(() -> {
    System.out.println("回调中执行的内容");
});
3）Spring 中典型回调：JdbcTemplate
| 优点      | 说明                          |
| ------- | --------------------------- |
| ✅ 解耦    | 调用者和实现者分离，实现更灵活             |
| ✅ 异步处理  | 特别适合异步任务完成后的通知              |
| ✅ 便于扩展  | 可随时更换、注入新的回调逻辑              |
| ✅ 框架可插拔 | Spring、Netty 等大量使用回调机制实现扩展点 |
| 问题     | 说明                |
| ------ | ----------------- |
| 回调地狱   | 多层嵌套回调导致代码难以阅读    |
| 难于调试   | 异步回调发生时难以定位问题     |
| 内存泄漏风险 | 回调引用未释放，导致对象无法 GC |
| 时序复杂   | 异步回调执行顺序不确定       |
Java 中为了缓解回调地狱，后来引入了 CompletableFuture、Reactor、RxJava 等响应式框架。
1）CompletableFuture（Java 8 原生）
🌟 特点：
支持链式异步编程
提供丰富的组合方法（thenApply, thenCompose, exceptionally 等）
使用线程池异步执行，不阻塞主线程
📌 示例：
CompletableFuture.supplyAsync(() -> loadUser(userId))
    .thenCompose(user -> CompletableFuture.supplyAsync(() -> loadProfile(user)))
    .thenCompose(profile -> CompletableFuture.supplyAsync(() -> loadPosts(profile)))
    .thenAccept(posts -> System.out.println("Posts: " + posts))
    .exceptionally(ex -> {
        log.error("出错了", ex);
        return null;
    });
🔁 效果等同于嵌套回调，但通过链式表达，逻辑清晰，异常统一处理。
2）Reactor（Project Reactor）——Spring WebFlux 所用响应式框架
🌟 特点：
支持响应式流（背压 Backpressure）
Mono（0-1个元素），Flux（0-N个元素）
非阻塞、异步、可组合
与 Spring WebFlux 深度整合
📌 示例：
Mono.just(userId)
    .flatMap(this::loadUser)
    .flatMap(this::loadProfile)
    .flatMap(this::loadPosts)
    .subscribe(posts -> System.out.println("Posts: " + posts),
               error -> log.error("出错了", error));
 🌱这是处理异步链最推荐的方式，语义清晰、性能优异，Spring 项目首选。
3）RxJava（ReactiveX for Java）
🌟 特点：
功能极强的响应式框架
支持 Observable, Single, Flowable 等多种类型
丰富的操作符链（map, flatMap, filter, zip 等）
非常适合数据流转换和异步流程控制
Observable.just(userId)
    .flatMap(this::loadUser)
    .flatMap(this::loadProfile)
    .flatMap(this::loadPosts)
    .subscribe(posts -> System.out.println("Posts: " + posts),
               error -> log.error("Rx出错了", error));
🔍 RxJava 常用于 Android 或对响应式链路要求极高的服务端场景。
| 特性        | `CompletableFuture` | `Reactor`       | `RxJava`       |
| --------- | ------------------- | --------------- | -------------- |
| 是否原生      | ✅ JDK8+ 原生          | ❌ 第三方库          | ❌ 第三方库         |
| 多值流支持     | ❌（仅单值）              | ✅（Flux）         | ✅（Observable）  |
| Spring 支持 | ☑️（较少）              | ✅（WebFlux 官方支持） | ☑️（社区集成）       |
| 背压支持      | ❌                   | ✅               | ✅（通过 Flowable） |
| 编码简洁度     | 中等                  | ✅               | ✅              |
| 学习曲线      | 简单                  | 中等              | 较高             |


构建Agent应用
基本定义：AI Agent 应用是指围绕一个或多个任务目标，结合大语言模型（LLM）+ 工具使用能力（Tool Use）+ 记忆 + 状态管理，持续进行感知—思考—行动的系统。
关键特征：
| 特性         | 说明                       |
| ---------- | ------------------------ |
| **自主性**    | 可根据目标自主规划和执行任务           |
| **可调用工具**  | 如搜索、数据库、接口、脚本等           |
| **状态管理**   | 能记住历史、上下文或中间状态           |
| **链式推理能力** | 可多轮调用工具并分析结果再决策          |
| **模块可组合性** | 拆分为感知 / 思考 / 行动 / 反馈多个模块 |
架构组成：
┌────────────┐
│  用户请求  │
└────┬───────┘
     ↓
┌────────────┐
│   控制器    │  → Spring Boot 接口
└────┬───────┘
     ↓
┌────────────┐
│  Agent 管理 │  → AgentRegistry / 执行器
└────┬───────┘
     ↓
┌────────────────────────┐
│   Reasoner（思考逻辑）   │ ← Prompt + Planner
└────┬──────┬────────────┘
     ↓      ↓
┌──────┐ ┌──────────────┐
│Tool1 │ │ Memory模块    │
└──────┘ └──────────────┘

可集成的 Java 工具 & 框架：
| 类别          | 建议工具                                      |
| ----------- | ----------------------------------------- |
| 大模型调用       | OpenAI SDK, Gemini SDK, Azure OpenAI      |
| SSE 响应      | `Flux<String>` + Spring WebFlux           |
| JSON 工具     | Jackson / Gson                            |
| Prompt 模板引擎 | FreeMarker / custom DSL                   |
| Tool 执行框架   | Java SPI、Spring Bean 注册、Function Registry |
| 任务调度        | Reactor 流控制，异步链式执行                        |

推荐封装方向（通用 Agent SDK）：
1）AgentExecutor：控制主流程
2）ToolRegistry：注册与执行工具
3）Planner：根据上下文生成 prompt
4）ToolCallParser：解析 <tool_call> JSON
5）MemoryStore：存储上下文 / 历史信息
6）FunctionCallSupport：支持 Function Calling 接口



Embedding API 的原理
1-什么是 Embedding（向量化表示）
1)Embedding 就是把一段文本（或者图片、音频等）转换成一个高维向量（一般几百～几千维）。
2)每个维度代表了语义空间中的某种特征。相似的文本会被映射到语义空间中的相近位置。
2-Embedding API 的原理
背后是一个 深度神经网络模型（通常是 Transformer）。
模型输入：文本序列 → Token 化（如 BPE、SentencePiece）。
模型输出：句子/词的向量表示（hidden states 聚合）。
常见聚合方式：[CLS] token 向量（BERT 类模型）。平均池化（mean pooling）。加权池化（attention pooling）。
Embedding 维度：OpenAI Ada v2: 1536 维、Gemini Embeddings: 768/1024/3072 等不同规格
Embedding 的语义空间是 度量空间，常用相似度度量：余弦相似度 (cosine similarity)、向量内积 (dot product)、欧式距离 (L2 norm)
3-主要应用场景
语义搜索：把文档向量化存到数据库，查询时向量化查询语句，找最相似的文档。
推荐系统：向量检索相似物品或用户。
聚类/分类：基于向量的聚类、KNN 分类。
RAG（检索增强生成）：LLM 之前先用 embedding 做知识检索。
4-调用方式总结
输入：字符串 / 批量字符串
输出：向量数组（通常是浮点数数组）
使用方式：
1）将文本向量化并存储在向量数据库（如 Pinecone, Weaviate, Milvus, FAISS）
2）查询时先向量化 query，再检索最相似的向量
3）把检索到的内容交给 LLM 作为上下文
⚡ 总结一句话：
Embedding API 的本质是 把语义信息映射到向量空间，调用方式很简单（输入文本 → 输出浮点向量），关键在于 如何在下游应用中利用向量（搜索、推荐、RAG 等）。


1. Cosine Similarity（余弦相似度）
含义：计算两个向量之间的夹角余弦值。范围在 [−1,1][−1,1]。
特点：忽略向量的绝对大小，只关注方向。适合 文本向量，因为 embedding 主要表示语义方向。如果向量归一化（norm=1），cosine 与 dot product 等价。
应用场景：NLP 相似度搜索（Sentence Embedding, RAG 检索）。用户画像推荐（方向比大小更重要）。
2. Dot Product（点积 / 内积）
含义：衡量两个向量的投影大小。
特点：与向量的 长度 + 方向 都有关。向量越长，点积值可能越大 → 对未归一化 embedding 有影响。通常 embedding 模型（如 OpenAI, BERT）会生成归一化向量，此时 dot product ≈ cosine。
应用场景：推荐系统（用户向量 × 物品向量）。Transformer attention 机制（Q·K^T）。
3. L2 Distance（欧几里得距离）
含义：传统的几何距离。
特点：考虑了向量大小差异。与 Cosine 不同，长度越大的向量更可能被区分开。需要保证 embedding 维度空间分布均匀，否则可能不稳定。
应用场景：图像搜索（pixel/feature 直观差异）。数值型数据聚类（K-means 默认使用 L2）。

向量归一化：把一个向量缩放到 长度（范数）为 1，但保持方向不变。


🔑 一、为什么需要向量数据库？
传统数据库（MySQL、Postgres）主要支持 结构化数据 查询（例如 id=1、name LIKE '%chatgpt%'）。但是在 AI 语义搜索、推荐系统、RAG（检索增强生成） 场景中，数据是 非结构化的：
文本 → embedding 向量（OpenAI Embeddings / BERT）
图片 → embedding 向量（CLIP）
音频、视频 → embedding 向量
👉 这些 embedding 向量通常是 几百到几千维的浮点向量。我们需要根据“相似度”来查询（语义最近的内容），而不是传统的 SQL 精确匹配。这就需要 向量数据库。
🔑 二、向量数据库的核心功能
1）存储
存储高维向量（通常 128～4096 维）以及其关联的 metadata（文档 ID、标题、URL 等）。
2）相似度搜索（ANN: Approximate Nearest Neighbor）
给定一个查询向量 q，找到数据库中最相似的 k 个向量。常见度量方式：Cosine 相似度、L2 距离（欧氏距离）、Dot product
3）索引结构
因为向量很大、数据量很高，需要专门的索引：IVF（Inverted File Index）、HNSW（Hierarchical Navigable Small World Graph）、PQ（Product Quantization）
能够在百万 / 十亿向量库里快速检索。
4）分布式能力
支持水平扩展（Sharding、Replica）、高可用（HA）、自动容错。
5）混合查询（Hybrid Search）
向量 + 结构化条件


https://github.com/milvus-io/milvus
Milvus 本身是完全无状态的，因此可以借助 Kubernetes 或公共云轻松扩展。
Milvus的最关键的三项任务--搜索、数据插入和索引/压实。
定义了collection的维度后，如何存储？
Collection相当于sql中的数据库表
密集向量：理解语义。稀疏向量：精确匹配。二进制向量：查重 / 快速匹配


长期记忆
1、什么时候存（事件驱动 / 阈值）
1）事件驱动（立即入库）
用户显式“收藏/置顶/标记重要”
会话结束（EndSession Hook）
触发域事件：下单、退款、绑定邮箱等
2）阈值策略（满足条件才存）
重要性得分 ≥ S_THRESHOLD（关键实体、长期偏好、联系方式、配置类信息）
会话窗口累计字数 ≥ LEN_THRESHOLD
语义新颖度（与已存最相似项的余弦相似度 < NOVELTY_THRESHOLD）
频次累计（同类信息出现 ≥ N 次）
2、长期记忆检索优先级（最新优先 / 语义优先）
1）组合打分：score = w_sem * cosineSim + w_rec * recencyBoost + w_src * sourceBoost
cosineSim：向量相似度
recencyBoost = exp(-Δt / τ)（越新越高）
sourceBoost：来自“显式标记/系统事件”可加权
2）典型权重：w_sem=0.6, w_rec=0.3, w_src=0.1（可配置/AB 测试）


SLO：内部设定的服务可靠性目标（工程指标）。
SLA：对客户承诺的服务可靠性协议（商业合同）。


工作流Agent与推理Agent
| 维度       | 工作流 Agent          | 推理 Agent           |
| -------- | ------------------ | ------------------ |
| **驱动力**  | 任务流程图 / 状态机 / DAG  | 思维链 / 规划 / 自适应决策   |
| **执行方式** | **预定义步骤**（固定顺序）    | **动态生成步骤**（临时规划）   |
| **稳定性**  | 高（结果可控，易测试）        | 中等（依赖模型推理，结果波动大）   |
| **灵活性**  | 低（难处理未知情况）         | 高（可应对新问题）          |
| **工具使用** | 工具调用点通常预先定义        | 工具调用点由模型推理决定       |
| **适合任务** | 流程型任务、批处理、流水线      | 问题求解、探索、策略规划       |
| **例子**   | ETL、审批流、文档生成、API编排 | 复杂问答、自动 Debug、决策优化 |

好 👍，我来给你系统性、深入地梳理一下大模型 **推理机制**（Reasoning Mechanisms），包括你提到的 **CoT、ToT、Self-Consistency、Reflexion、ReAct**，它们基本代表了近几年 LLM 推理研究的核心脉络。

---

# 1️⃣ Chain-of-Thought (CoT, 思维链)

### 🔹 核心思想

* 在回答复杂问题时，模型不是直接给答案，而是先“写出中间推理步骤”。
* 类似人类考试：**先写草稿推理过程，再写最终答案**。

### 🔹 实现方式

* 通过提示语（prompting）诱导模型输出推理链。
* 例如：

  ```
  Q: 如果商店里有 3 个苹果，我买走 2 个，还剩多少？
  A: 让我们一步步思考。
  1. 初始数量 = 3
  2. 买走 = 2
  3. 剩余 = 3 - 2 = 1
  答案：1
  ```

### 🔹 优缺点

* ✅ 优点：显著提升复杂推理（数学、逻辑题）效果。
* ❌ 缺点：如果推理链错误，最终答案也错误（garbage in, garbage out）。

---

# 2️⃣ Tree-of-Thought (ToT, 思维树)

### 🔹 核心思想

* 在 CoT 的基础上引入 **分支探索**：不是只走一条推理路径，而是像树一样，探索多种可能的推理路径。
* 模型可以在多个推理分支中选择最优路径，甚至回溯。

### 🔹 实现方式

* 将问题分解为 **状态（state）** 和 **操作（step）**。
* 使用搜索算法（DFS、BFS、启发式搜索）探索推理树。

### 🔹 示例

解谜题时，模型可能探索：

* 路径 A → 结论
* 路径 B → 结论
* 对比后，选最优解。

### 🔹 优缺点

* ✅ 优点：更鲁棒，能避免单一推理链错误。
* ❌ 缺点：计算量大，推理速度慢，需要裁剪（pruning）。

---

# 3️⃣ Self-Consistency (自洽性)

### 🔹 核心思想

* 多次运行 CoT，得到多个推理链 → 取 **多数票答案**。
* 类似“集体智慧”：一个模型回答可能出错，但多次回答后选出最常见的结果，准确率更高。

### 🔹 实现方式

1. 用相同问题，生成 N 条推理链。
2. 收集最终答案。
3. 投票选出出现频率最高的答案。

### 🔹 示例

* 第一次推理：答案 1
* 第二次推理：答案 3
* 第三次推理：答案 1
  → 多数票是 1，取结果 1。

### 🔹 优缺点

* ✅ 优点：在推理问题（尤其是数学）中显著提升准确率。
* ❌ 缺点：增加推理成本（多次采样）。

---

# 4️⃣ Reflexion（反思机制）

### 🔹 核心思想

* 模型不仅推理，还能 **回顾并修正自己的错误**。
* 类似人类“自我反思”：做错题后分析错误原因，下次改进。

### 🔹 实现方式

* 增加一个 **反思循环**：

  1. 初次回答。
  2. 检查答案是否合理。
  3. 如果发现错误 → 重新推理。

* 常结合 **记忆机制**，模型能保留错误经验并改进。

### 🔹 示例

```
Q: 17 * 18 = ?
A1: 306  （错误）
Reflexion: 我检查一下。17*18 = 17*(20-2) = 340 - 34 = 306，哦对了其实是对的。
→ 最终答案：306
```

### 🔹 优缺点

* ✅ 优点：增强模型的自校正能力，减少“胡说八道”。
* ❌ 缺点：反思能力依赖 prompt 设计，不一定真的能发现错误。

---

# 5️⃣ ReAct（Reason + Act，自适应规划）

### 🔹 核心思想

* 结合 **推理（Reasoning）** 和 **行动（Acting）**。
* 模型一边推理，一边调用工具（搜索、计算器、数据库）。
* 类似“思维链 + 工具使用”。

### 🔹 实现方式

* 输出格式通常分为两部分：

  ```
  Thought: 我需要知道 X 的定义
  Action: 搜索("X")
  Observation: X 的定义是...
  Thought: 好，现在我可以回答问题了
  Answer: ...
  ```

### 🔹 示例

问：法国总统的年龄是多少？

* Reasoning: 我需要法国总统的出生日期
* Action: 调用搜索工具 → 得到“埃马纽埃尔·马克龙，1977年出生”
* Reasoning: 今年是 2025 → 2025 - 1977 = 48
* Answer: 48 岁

### 🔹 优缺点

* ✅ 优点：能动态使用外部工具，提升事实性和能力边界。
* ❌ 缺点：需要可靠的工具调用接口，执行链复杂，容易出错。

---

# 🔎 总结对比

| 机制                   | 特点           | 适用场景           |
| -------------------- | ------------ | -------------- |
| **CoT**              | 单链推理，显式思维步骤  | 数学题、逻辑题        |
| **ToT**              | 多路径探索，搜索最优   | 复杂推理、规划        |
| **Self-Consistency** | 多次采样投票，增强稳定性 | 数学、QA、逻辑题      |
| **Reflexion**        | 自我反思 + 修正错误  | 长链推理、Debug、对话  |
| **ReAct**            | 推理 + 工具使用    | 知识问答、检索、API 调用 |

---

👉 可以这么理解：

* **CoT**：像学生“写草稿”
* **ToT**：像团队“头脑风暴”
* **Self-Consistency**：像民主“多数投票”
* **Reflexion**：像老师“批改并纠错”
* **ReAct**：像科学家“推理 + 查资料 + 实验”


--------------------------------------------------------------------------------------------------------------------------------------------




















































