# MVP 实施计划：聊天式短线交易 Harness 内核

## 1. MVP 目标

本阶段先做系统内核，不做 UI，不接真实交易，不接同花顺或开盘啦实时接口。MVP 的目标是验证一件事：用户能通过文字对话驱动系统完成策略维护、盘前计划、盘中模拟观察、盘后复盘和策略更新提案，并且所有过程都有结构化状态、策略上下文和审计记录。

MVP 的能力等级定位为 L0-L2：

- L0：整理历史经验，形成策略资产。
- L1：生成盘前计划和盘后复盘。
- L2：支持模拟交易记录和纸面账户归因。

MVP 明确不做：

- 不自动实盘下单。
- 不绕过人工确认。
- 不把 agent 的单次观点直接写成正式策略。
- 不依赖简单 RAG 作为主要加载机制。
- 不先开发 UI。

## 2. 第一阶段交付物

### 2.1 核心目录

建立系统的最小目录约定：

```text
data/
  raw/
  extracted/
  assets/
  runtime/
  strategy_versions/
  controls/
schemas/
```

`data/` 保存运行时和策略资产数据，`schemas/` 保存核心对象的 JSON Schema，`harness/` 保存系统设计和工程规范。

### 2.2 核心 Schema

第一批必须定义这些对象：

- `source_document`：所有原始输入的统一来源记录。
- `atomic_insight`：从复盘、笔记、交易记录中抽取出的最小经验单元。
- `strategy_card`：可被 agent 加载的策略卡片。
- `risk_rule`：独立于策略的风控规则。
- `context_pack`：每次任务执行前编译出的上下文包。
- `conversation_task`：从用户对话转成的结构化任务。
- `postmarket_review`：盘后复盘记录。
- `order_intent`：L3 以后使用的交易意图草案，MVP 只定义不执行。

### 2.3 历史经验冷启动

从 `daily_review_files/` 中选择一张图片作为第一批样本，抽取 20-50 条高质量 `atomic_insight`。先不要追求全量自动化，重点验证 schema 是否能表达真实经验。

冷启动流程：

1. 记录图片为 `source_document`。
2. 按主题拆成若干 `note_section`。
3. 从每段中提取 `atomic_insight`。
4. 把若干相关 insight 编译成 3-5 张 `strategy_card`。
5. 人工校对术语、适用条件和风险条件。

### 2.4 对话任务路由

实现一个最小的文本入口，先支持以下任务：

- `create_premarket_plan`：生成盘前计划。
- `add_strategy_note`：新增用户文字策略。
- `review_trade`：复盘一笔交易。
- `review_day`：复盘一天。
- `propose_strategy_update`：提出策略更新建议。
- `assess_intraday_setup`：盘中模拟判断某个机会是否符合计划。

输入可以先用命令行或脚本参数，不需要 UI。重点是每条用户消息都必须转成 `conversation_task`，并生成或引用对应的 `context_pack`。

### 2.5 盘前/盘后闭环

MVP 的第一条完整闭环：

```text
用户输入市场状态和候选方向
-> 系统生成 conversation_task
-> 编译 context_pack
-> 输出盘前计划
-> 用户盘后输入实际走势和交易结果
-> 系统生成 postmarket_review
-> 系统提出策略更新提案
-> 人工确认是否进入 strategy_versions
```

## 3. 核心模块设计

### 3.1 Asset Store

负责读写结构化资产，包括经验、策略、风控、失败模式和术语。

MVP 可以先使用文件系统和 JSON/YAML 文件，不急着上数据库。文件系统更适合早期人工检查和版本管理。

### 3.2 Conversation Router

负责把用户文字转成结构化任务。

输入：

```text
今天帮我做盘前计划，市场感觉还在修复，但中位股亏钱效应没有完全消失。
```

输出：

```yaml
task_type: create_premarket_plan
requires_market_snapshot: true
requires_strategy_context: true
trading_level: L1
```

### 3.3 Context Compiler

负责根据任务和市场状态加载正确策略，不走简单全文检索。

加载顺序：

1. 常驻纪律和系统边界。
2. 当前交易能力等级。
3. 当前市场状态。
4. 与任务匹配的 active 策略卡片。
5. 风控规则。
6. 失败模式。
7. 必要的来源证据引用。

### 3.4 Review Engine

负责盘后归因。

必须区分：

- 策略错误：策略本身不适用或规则缺陷。
- 执行错误：策略有效，但执行偏离。
- 环境错误：市场状态判断错误。
- 随机波动：暂不足以修改策略。

### 3.5 Strategy Maintenance

负责处理用户新增策略和 agent 的复盘优化提案。

正式策略更新必须经过：

1. 生成 `strategy_update_request`。
2. 定位影响的策略和风控。
3. 冲突检查。
4. 样本或理由绑定。
5. 生成候选版本。
6. 人工确认后发布。

## 4. 开发顺序

### Step 1：建立目录和 schema

完成 `data/README.md` 和 `schemas/*.schema.json`。

验收标准：

- 每个核心对象都有 schema。
- schema 能表达来源、状态、版本、证据和审核信息。
- schema 之间通过 id 引用，而不是复制大段文本。

### Step 2：整理第一批策略种子

从一张历史图片中抽取第一批 `atomic_insight`，并人工校对。

验收标准：

- 至少 20 条 insight。
- 每条 insight 都有来源、原文摘录、规范化表述、适用条件和类型。
- 至少能形成 3 张策略卡片或风控规则。

### Step 3：实现最小任务路由

写一个本地脚本或 CLI，将用户输入转成 `conversation_task`。

验收标准：

- 能识别盘前计划、策略新增、交易复盘、盘中模拟判断。
- 每个任务都有 `task_id`、`task_type`、`status`、`required_context` 和 `output_contract`。

### Step 4：实现 context_pack 编译

根据任务和市场状态，从策略资产中选择需要加载的内容。

验收标准：

- 盘前计划只加载 active 策略和风控。
- pending 策略不能用于交易建议。
- 风控规则优先于机会策略。

### Step 5：跑通盘前/盘后样例

使用人工输入的市场状态和候选股，生成盘前计划；盘后输入实际结果，生成复盘和策略提案。

验收标准：

- 输出能说明依据哪些策略和风控。
- 复盘能区分策略错误、执行错误、环境错误。
- 策略优化进入审核队列，不直接生效。

## 5. MVP 验收清单

- 是否存在完整的核心 schema。
- 是否能把历史图片经验抽取成结构化 `atomic_insight`。
- 是否能把 insight 编译成 `strategy_card` 和 `risk_rule`。
- 是否能把用户文字转成 `conversation_task`。
- 是否能按任务生成 `context_pack`。
- 是否能输出盘前计划。
- 是否能记录模拟交易和盘后复盘。
- 是否能生成策略更新提案。
- 是否能阻止未确认策略直接进入交易建议。
- 是否能把真实交易相关请求拦截为 `order_intent` 或研究建议。

## 6. 下一步建议

下一步从 Step 1 开始：先建立 `data/` 目录说明和第一批 `schemas/` 文件。Schema 稳定后，再从 `daily_review_files/20240609.png` 抽取第一批策略种子。
