# Prompt 迭代日志 · LMAPI 知识网页

> 大模型与自然语言处理 · 第 2 次课实验 · 子任务 F
> 子任务：**Provider Portability —— 能跑 ≠ 语义相同（OpenAI / Claude / Gemini 同一段 SDK 代码的行为差异，含兼容层陷阱）**
> 参考官方文档（一手来源）：
> - https://ai.google.dev/gemini-api/docs/openai （Gemini OpenAI 兼容层）
> - https://ai.google.dev/gemini-api/docs/text-generation （Gemini 原生）
> - https://docs.anthropic.com/en/api/messages （Anthropic Messages API）
> - https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html （vLLM 兼容服务）
> - https://platform.openai.com/docs/api-reference/chat/create （OpenAI 参考）

---

## 【第 1 版 Prompt】（原版 · 单语版规格）

```text
# Task
做一个可交互、视觉质量高的单页知识网页，讲解子任务 F「Provider Portability」：
同一段 OpenAI SDK 代码，把 base_url 换到 Claude / Gemini / vLLM 兼容端点后，
参数被改名、丢弃、报错或静默忽略，行为与语义各不相同。
受众：已学过 Transformer / Python 的大四学生。

# Constraints
- 白色基础界面，内容清晰易懂
- 每页只讲清 1 个概念，避免堆砌
- 必须含 1 个可交互 demo 或动画
- 引用厂商官方一手文档，禁止编造

# Data
- 核心概念：200 只证明端点存在，不证明语义相同
- 对比对象：OpenAI（参考）/ Anthropic Claude / Google Gemini / vLLM
- 三类失败模式：直接报错 / 静默忽略 / 语义漂移

# Output
- 单文件 index.html（CSS/JS 内联）
```

### Hermes 输出问题（原版暴露的问题）

1. **语言缺失**：输出为单一中文版本。课程/用户要求「中英双语两个模块、右上角可切换、选中文显示中文版」时，原规格根本没有定义语言层，导致整页返工重排。
2. **模块边界缺失**：规格只写了「一个可交互 demo」，没写「认识 → 使用/对比 → 验证」的结构；模型把「介绍」与「对比使用」混在同一信息流里，交互点（移植台、差异矩阵、测验、清单）全靠自由发挥。
3. **数据与指令混在一起**：官方文档 URL 只散落在描述里，没有进 `# Data` 区，模型引用时存在「只信博客、不贴一手出处」的倾向。

---

## 【第 2 版 Prompt】（中英文修改版 · 改了哪部分规格）

```text
# Task
同上，但输出必须是「中英双语双模块」页面：选择中文则显示中文版，选 EN 则显示英文版。

# Constraints（指令区，非数据）
- 语言层：每个文本节点同时写 <span class="z">中文</span> 与 <span class="e">English</span> 两套；
  body.lang-zh 默认中文，lang-en 显示英文；右上角「中文 | EN」切换按钮 + localStorage 记忆
- 模块结构（信息即顺序）：01 问题（三类失败模式）→ 02 移植台（端点标签页 + 代码 + 差异清单）
  → 03 差异矩阵（筛选 + 行展开详解）→ 04 陷阱测试（判分 + 解析出处）→ 05 迁移清单（进度 + 重置）
  → 06 结论（三句话）→ 07 一手资料（官方链接）
- 交互：每个模块必须可操作；可点元素用 button，禁止僵尸按钮
- 数据保真：每个差异点必须可追溯到官方文档链接，禁止二手博客

# Data（数据区，与指令分离）
- 已核对事实（2026-10）：
  · Gemini 兼容层：base_url=https://generativelanguage.googleapis.com/v1beta/openai/；
    reasoning_effort ↔ thinking_level/thinking_budget；白名单外参数静默忽略
  · Gemini 原生：system_instruction 顶层；max_output_tokens；3.6+ 忽略 temperature/top_p/top_k
  · Anthropic：max_tokens 必填（≥1024 级）；system 顶层；usage 为 input/output_tokens；
    frequency_penalty 等不存在（拒绝）
  · vLLM：模型无 chat template 则 chat 报错；extra_body 传 top_k/guided_json 等；
    repetition_penalty ≠ frequency_penalty
- 官方文档 URL：见本文件头部「参考官方文档」

# Output
- 单文件 index.html（CSS/JS 内联，白底 + 厂商色徽章）
- 同目录交付 AGENTS.md、PROMPT_LOG.md
```

### 效果变化（第 2 版相比第 1 版）

1. **语言闭环**：右上角切换「中文 / EN」，选中文即中文版、选 EN 即英文版，刷新后记住语言（localStorage）。
2. **叙事完整**：页面从「问题（认识）→ 移植台/矩阵（使用与对比）→ 测验/清单（验证）」逐层推进，每个模块独立可操作：6 个端点标签页、12 行差异矩阵可筛选可展开、6 题测验即时判分、10 项清单带进度条。
3. **可追溯**：每一处差异结论都挂了一手文档链接（页尾「07 一手资料」6 张官方卡片），杜绝无出处断言。

---

## 【结论】

> 这个知识点（Provider Portability）最该用哪一层解决？ → **☑ Eval**

理由：**「能跑 ≠ 语义相同」的核心是行为验证问题**。Prompt 层只能约束生成质量，Schema 层只约束请求形状，Runtime 层解决执行环境——它们都无法回答「换端点后行为是否等价」。判断一次迁移是否成功，必须用固定 test case 对比输出行为（格式、长度、失败模式），而不是看 HTTP 200。这与本页「陷阱测试 Q5」和「迁移清单第 6 条」的结论一致。

> 一句话总结学到的 Prompt Engineering 方法：

**把「语言、模块边界、交互清单、数据来源」全部写进规格的 Constraints（指令区），并坚持指令与数据分离（# Constraints / # Data 分块），模型的输出才可预期、可验收——先规格化，再让模型生成，而不是「试三次觉得差不多」。**
