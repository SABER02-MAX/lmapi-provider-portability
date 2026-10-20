# 项目：LMAPI 知识网页 - Provider Portability（子任务 F）

## Task
做一个可交互、视觉质量高的单页知识网页，讲解子任务 F「Provider Portability」：
同一段 OpenAI SDK 风格的代码，把 `base_url` / 端点换到 Anthropic Claude、Google Gemini、
vLLM 兼容端点、Azure OpenAI 后，参数被**改名、丢弃、报错或静默忽略**，行为与语义各不相同
（能跑 ≠ 语义相同）。

## Constraints（指令区，非数据）
- 受众：已学过 Transformer / Python 的大四学生
- 每节只讲清 1 个概念，避免堆砌；信息顺序即叙事：认识问题 → 对比使用 → 验证迁移
- 必须含 ≥1 个可交互 demo 或动画（本页实现：移植台端点标签页 / 差异矩阵筛选与行展开 / 陷阱测试判分 / 迁移清单进度）
- 引用官方一手文档，禁止编造；每个差异结论必须可追溯到文末「一手资料」链接
- 页面为「中英双语双模块」：每个文本节点同时提供 `.z`（中文）与 `.e`（英文）两套；
  `body.lang-zh` 默认中文、`lang-en` 显示英文；右上角「中文 | EN」切换 + localStorage 记忆，选择中文则显示中文版
- 白色基础界面、内容清晰易懂、响应式适配桌面与移动端；可点元素用 `button`，禁止僵尸按钮
- 数据保真：所有参数名、行为差异、默认值均来自厂商文档核对结果（2026-10），不目测、不凑整、不编造

## Data（数据区，与指令分离）
- 核心概念定义：
  - Provider Portability = 同一段应用代码在不同模型提供商 / 兼容端点之间迁移时，行为语义不一致的问题
  - 三类失败模式：直接报错（严格型，如 Anthropic 拒未知参数）/ 静默忽略（兼容层白名单外参数被丢弃）/ 语义漂移（同名参数含义不同）
  - 「OpenAI 兼容」只承诺请求格式相似，不承诺采样、默认值、计费、模型行为相同
- 已核对差异要点（2026-10）：
  - Gemini OpenAI 兼容层：只改 api_key / base_url / model 三行；`reasoning_effort` ↔ `thinking_level`/`thinking_budget`；白名单外参数静默忽略；Gemini 专属字段走 `extra_body={"google":{...}}`
  - Gemini 原生：`system_instruction` 顶层；`max_tokens` → `generation_config.max_output_tokens`；`messages` → `input`/`contents`；Gemini 3.6+/4 忽略自定义 temperature/top_p/top_k（返回弃用警告头）
  - Anthropic：`max_tokens` 必填；`system` 顶层参数；role 只有 user/assistant；无 penalty/seed/response_format（严格拒绝）；usage 为 input_tokens/output_tokens
  - vLLM：模型 tokenizer 必须自带 Jinja2 chat template 否则 chat 报错；`extra_body` 传 top_k/min_p/guided_json/guided_regex/guided_choice/repetition_penalty；`repetition_penalty` 与 OpenAI `frequency_penalty` 是不同算法
  - Azure OpenAI：`api_version` 必填；`model` 传部署名；新能力分批上线；内容过滤默认介入
- 参考素材路径（官方一手文档）：
  - https://ai.google.dev/gemini-api/docs/openai
  - https://ai.google.dev/gemini-api/docs/text-generation
  - https://docs.anthropic.com/en/api/messages
  - https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html
  - https://platform.openai.com/docs/api-reference/chat/create
  - https://learn.microsoft.com/en-us/azure/ai-foundry/openai/reference

## Output
- 单文件 `index.html`（CSS/JS 内联，白底 + 厂商色徽章，自包含可离线打开）
- 同目录交付 `AGENTS.md`（本文件）与 `PROMPT_LOG.md`（Prompt 迭代日志）
- 部署：GitHub Pages（独立仓库，根目录含 `index.html`），公网可访问
