# Agent-harness 基准测试：成本与准确性

一个小型、自包含的测试工具，用于通过三种 Agent 框架在您选择的模型上运行**相同的 Agent 任务套件**，使用**盲审 LLM 评判器**对答案进行评分，并并排报告**准确性、成本和延迟**。

由 TrueFoundry 构建，用于在一套跨系统企业任务上，将 [TrueForge](https://github.com/truefoundry/trueforge)（我们的开源 Agent 工具）与 **Claude 托管 Agent (CMA)** 进行比较；同时包含一个 **deepagents** (LangGraph) 分支。这里的所有内容都是我们实际运行的内容——没有隐藏的评分标准，没有针对单个任务的提示，每个框架都使用统一的提示词。您可以将其指向自己的数据集和模型，应该能复现我们结果的大致形态。

---

## 测量内容

该套件是一组**跨系统数据任务**，涉及三个以后端服务形式暴露为 MCP 服务器的服务：

- **pm** — 一个 Jira 风格的项目管理服务器（Issue、评论、组件、Wiki）
- **crm** — 一个 Salesforce 风格的 CRM（类似 SOQL 的查询、记录、Schema 描述）
- **file-server** — 一个 Drive 风格的文档存储（搜索、读取、元数据）

每个任务都会提出一个问题，需要**跨两个或三个系统连接数据**（例如：将未解决的支持工单与其所属的 CRM 账户以及背后的工程组件关联起来）。Agent 必须发现 Schema、找出关联关系、提取数据，并写出有根据的答案。

### 数据集

我们针对 [DevRev 的 Enterprise-Bench](https://x.com/devrev/status/2075815567458734468) 中的 **L1-L2 任务**进行运行，DevRev 已将其开源——包括数据集、评分标准以及他们的 [Harbor](https://github.com/harbor-framework/harbor) 评估工具（该数据集发布在 Harbor Hub 上）。我们**不会**在此重新分发他们的数据——请从 DevRev 的发布中获取，并将 `DATASET_DIR` 指向该目录。该工具期望每个任务具有以下结构：

```
dataset/
  <task-id>/
    prompt.txt              # 提供给 Agent 的任务指令
    tests/criteria.yaml     # 评判器依据的必需标准
```

任何符合此结构的套件都可以使用——此工具并非 DevRev 专用。

---

## 三个分支

| 分支          | 框架                             | Agent 运行方式                               | 工具                                         |
| ------------ | ------------------------------------- | ------------------------------------------------ | --------------------------------------------- |
| `tfy`        | TrueForge (TrueFoundry Agent Harness) | 通过 TrueForge 自己的 HTTP API (`/api/v1`) 进行会话 | 在 TrueForge 中注册的 MCP 服务器（按名称） |
| `cma`        | Claude 托管 Agent (beta)          | Anthropic 托管的会话 + 沙箱               | 附加到 Agent 的 `mcp_toolset`           |
| `deepagents` | deepagents (LangGraph)                | 本地 `create_deep_agent` 循环                   | 通过 `MultiServerMCPClient` 使用 MCP 工具          |

每个分支都获得**相同的系统提示词** (`prompts/system.md`) 和**相同的任务提示词**。该提示词是简短、与模型无关的指导——以正确的粒度回答，并使用所有相关数据——不包含任何特定于任务的细节。

---

## 工作原理

1.  **`setup.py`** — 使用附带的系统提示词和 MCP 服务器创建 CMA 环境/Agent/Vault。TrueForge 和 deepagents 在此处无需设置步骤：TrueForge 在创建每个会话时发送内联的 Agent 规范，而 deepagents 在运行时附加 MCP 工具。（TrueForge 确实需要在其侧配置三件事——模型提供商、MCP 连接器以及**沙箱提供商**——请参阅下面的 **TrueForge 设置**。）
2.  **`bench_matrix.py run-all`** — 运行每个 `{分支 × 任务 × 试验}` 单元。每个任务作为子进程运行，并带有硬性挂钟超时，在超时/崩溃/空答案时会重试。完全可恢复——重新运行以继续。答案存放在 `results/<arm>/t<trial>/<task>.md`；指标存放在 `results/matrix.jsonl`。
3.  **`judge.py`** — 根据其 `criteria.yaml` 对每个答案进行评分。评判器只能看到标准和答案，永远看不到参考值，也看不到是哪个分支产生的答案。只有当**每个**必需标准都满足时，任务才算通过（没有部分得分）。写入 `results/grades.jsonl`。
4.  **`aggregate.py`** — 将矩阵汇总为每个分支的准确性、成本和延迟。`Solved / 14` 是每次试验通过任务的平均数。写入 `results/summary.csv`。

### TrueForge 设置（复现所需）

`tfy` 分支与正在运行的 TrueForge 服务器（`TFY_BASE_URL`）通信。在运行之前，**在 TrueForge 内部**配置三件事——遗漏任何一项都会导致数据密集型任务失败：

1.  **一个模型提供商**，用于 `MODEL_TFY`（例如 `anthropic/claude-opus-4-8`、`zai/glm-5-2`）。
2.  **来自 `mcp_config.json` 的 MCP 服务器**，以**相同名称**注册为连接器。
3.  **一个沙箱提供商**（例如 Daytona）。这是人们经常遗漏的一项。基准测试配置启用了 `large_tool_response`，它会**将过大的查询结果卸载到沙箱文件**中，而不是让它们堆积在模型的上下文中。其中几个任务会扫描数万条记录；**如果没有沙箱，上下文会超出令牌预算，并且回合会因 `max_tokens breached` 而中止。**

Agent 配置由**适配器内联发送**（`bench_matrix.py` 中的 `run_tfy`）——您无需在 TrueForge 中设置——并且是产生以下结果的经过验证的配置：

| 设置                                  | 值                         | 原因                                                      |
| ---------------------------------------- | ----------------------------- | -------------------------------------------------------- |
| `iteration_limit`                        | **500**                       | 足够的工具使用步骤，以完整处理跨系统任务  |
| `dynamic_sub_agents`                     | **启用**                   | 将大型子任务分解给具有全新上下文的子 Agent |
| `sandbox`                                | **启用**                   | `large_tool_response` 卸载所需（请参阅上文 #3）  |
| `context_management.compaction`          | **启用，阈值 60 000** | 总结旧历史记录，以便长时间回合保持在预算之内    |
| `context_management.large_tool_response` | **启用**                   | 将过大的工具结果卸载到沙箱            |
| `generative_ui`                          | 禁用                      | 无头基准测试不需要                      |
| `ask_user_questions`                     | 禁用                      | 完全自主；绝不为输入暂停回合           |

**长时间回合。** 最重的任务会运行几分钟。TrueForge 通过 SSE 流式传输回合事件，在长时间回合中，该流可能会在回合结束前关闭（回合在服务器端继续运行）。适配器处理了这种情况：如果流在没有终止 `turn.done` 的情况下结束，它会**轮询回合直到其终止状态**，而不是记录空答案。如果您编写自己的客户端，请执行相同操作，否则长时间任务看起来会像偶发故障。

**严格的 MCP 服务器。** MCP 规范要求客户端发送 `Accept: application/json, text/event-stream`。如果您的 MCP 服务器严格强制执行此要求，而 TrueForge 的连接器不满足，则工具列表会失败；请使用接受 TrueForge 发送的标头的 MCP 服务器（或轻量级代理）。

### 成本模型

成本是根据提供商列表价格（`$/1M`）下的令牌数计算的，而非估算值：

```
cost = (uncached_input * in_rate
        + output        * out_rate
        + cache_read    * cache_read_rate
        + cache_write   * cache_write_rate) / 1e6
```

不同框架的令牌语义不同，因此 `aggregate.py` 会对其进行归一化（`INPUT_IS_TOTAL`）：对于 TrueForge 和 deepagents，`input` 是**总**提示词，缓存读取/写入是其子集（`uncached = input - cache_read - cache_write`）；对于 CMA，`input` 已经是未缓存计数。费率默认为 Opus 4.8；当某个分支运行不同模型时，可使用 `RATES_<ARM>`（请参阅 `.env.example`）按分支覆盖。

---

## 运行

```bash
# 1. 安装（核心分支 + 评判器）
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 2. 配置
cp .env.example .env            # 填写密钥、主机、模型
cp mcp_config.example.json mcp_config.json   # 您的三个 MCP 服务器 URL
source .env

# 3. 运行完整流水线
python setup.py
python bench_matrix.py run-all
python judge.py
python aggregate.py             # 打印表格，写入 results/summary.csv
```

使用 `HARNESSES=tfy python bench_matrix.py run-all`（或 `run tfy`）运行单个分支。

**deepagents** 分支会引入 LangGraph/LangChain，这可能与核心栈冲突。如果 pip 报错，请从 `requirements-deepagents.txt` 创建第二个虚拟环境，并在其中运行 `HARNESSES=deepagents python bench_matrix.py run tfy`；`results/` 目录是共享的，因此 `judge.py` / `aggregate.py` 可以获取其结果。

Docker：`docker build -t trueforge-bench . && docker run --env-file .env -v $PWD/results:/bench/results trueforge-bench`
（运行核心分支；deepagents 请参阅上文）。

---

## 我们的结果

DevRev L1-L2 套件包含 **14 个任务**，每个任务在全新的会话中运行 **n = 3 次试验**。`Solved / 14` 是每次试验通过任务的平均数（盲审评判器，全有或全无）；成本是按提供商列表价格计算的每次运行成本。

| 配置                    |  n  | Solved / 14 | 每次运行成本 | 每次运行令牌数 |
| -------------------------------- | :-: | :---------: | :--------: | :----------: |
| Claude 托管 Agent · Opus 4.8 |  3  |    10.7     |   $11.8    |    10.0M     |
| TrueForge · Opus 4.8             |  3  |    10.7     |    $8.6    |     3.7M     |
| TrueForge · GLM-5.2              |  3  |    11.7     |    $3.0    |     3.8M     |

在**相同模型 (Opus 4.8)** 上，TrueForge 和 Claude 托管 Agent 解决了相同数量的任务（各 10.7 个）——准确性相当——而 TrueForge 每次运行使用的令牌要少得多（3.7M 对比 10.0M），因此成本更低（$8.6 对比 $11.8，约低 27%）。在开放模型 **GLM-5.2** 上运行 TrueForge 可解决 11.7 / 14 个任务，每次运行成本为 $3.0——比 Claude 托管 Agent 低约 75%。每次运行数字会有一两个任务的波动，因此请复现其形态，而不是单个单元格，并接入您自己的模型和费率。

套件中还包含一个 **deepagents** (LangGraph) 分支，可以以相同方式运行。

---

## 文件

```
prompts/system.md            所有分支共用的单一系统提示词（每个分支完全相同）
mcp_config.example.json      您的三个 MCP 服务器 URL 的模板
setup.py                     创建 CMA 环境/Agent/Vault（TrueForge + deepagents 不需要）
bench_matrix.py              运行 {分支 × 任务 × 试验}；可恢复，带超时保护
judge.py                     仅基于标准的盲审评分器
aggregate.py                 准确性 + 成本 + 延迟 -> summary.csv
requirements.txt             核心分支 + 评判器
requirements-deepagents.txt  deepagents 分支（单独的虚拟环境）
Dockerfile                   核心流水线
.env.example                 所有配置
```

## 方法

- **相同的提示词，相同的任务。** 每个分支都获得相同的系统提示词和任务提示词，并且每个答案都根据该分支的回复进行评分。
- **盲审、严格的评判器。** 评判器只能看到标准和答案——永远看不到参考值或产生答案的分支；只有当满足每个必需标准时，任务才算通过（没有部分得分）。
- **列表价格成本。** 根据提供商列表价格下的令牌数，针对每个框架的令牌报告进行归一化，按单元计算。
- **数据集。** DevRev 的 Enterprise-Bench；我们指向他们的发布版本，并不随附该数据集。
- **试验。** 报告多次试验的平均值并比较分布，而不是单个单元格。