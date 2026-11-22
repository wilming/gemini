# Build with Gemini · Track 5 — 实验指南（增强智能体优先应用）

使用 **Antigravity 2.0** 和 **ADK** 在 Google Cloud 上装备、观测、加固并优化企业级多智能体系统。

---

## 概述

一家创意机构构建了一个名为 **Agentic Pitch Generator（智能体提案生成器）** 的初始原型，用于将产品简报转化为营销活动提案。该初始原型运行一个四智能体并行工作流（`Creative Director`、`Brand Strategist`、`Copywriter` 和 `Visual Director`），具备 `LoopGuard` 循环保护和 Cloud Storage 产物持久化功能，但仍缺乏品牌指南技能、自动化合规评估、远程微服务隔离、BigQuery 多模态分析、安全护栏以及 Token 经济学成本控制。随着客户需求的增长，该机构需要一个能够安全、可审计且具成本效益地扩展的企业级多智能体平台。

在本实验中，您将使用 **Google Agent Development Kit (ADK)** 在 **Gemini Enterprise Agent Platform** 上演进该初始应用。在 **Antigravity 2.0** 中，您将与 Gemini 作为 AI 结对程序员协作，通过自然语言提示词引导 Gemini 在四个递进的工程模块中重构、扩展并加固代码库。

在实验第一步中，您将使用 Antigravity 审查 Agentic Pitch Generator 源代码（也可在 GitHub 上查看：`https://github.com/GoogleCloudPlatform/devrel-demos/tree/main/codelabs/bwg002-devrel-2`）。

### 实验目标
在本实验中，您将学习如何执行以下任务：
1. **装备并扩展专家智能体团队**：通过可复用的品牌技能（Skills）、自动化合规评估以及基于 **Agent2Agent (A2A)** 协议的远程微服务。
2. **观测并审计智能体行为**：将 Cloud Storage 中的多模态资产关联到 BigQuery 对象表（Object Tables），并使用 `AI.SCORE` 评估品牌漂移（Brand Drift）。
3. **加固并保护流水线**：通过生命周期钩子（Lifecycle Hooks）、敏感 PII 数据脱敏以及人工审批（Human-in-the-Loop, HITL）关卡。
4. **通过 Token 经济学（Tokenomics）优化规模化性能**：使用记忆压缩、滑动窗口历史裁剪以及 SHA-256 提示词缓存。

### 先决条件
本大师课建立在以下基础智能体概念之上：
- 自主智能体如何基于指令、输入和工具进行推理以达成目标。
- 智能体如何调用外部 Python 函数和 API 执行现实世界的操作。
- 区分持久的角色指令与动态的单次请求简报。
- 区分短期会话状态与 Gemini Enterprise Agent Platform Memory Bank 中的跨会话记忆。
- 标准的 Agent Development Kit 项目结构与 CLI 入口点。
- 使用依赖注入的 Mock 在本地测试智能体，以及在 Cloud Run 上运行容器化服务。

---

## 环境准备

在本实验中，您将获得一个临时 GCP 环境以及一个远程虚拟桌面，所有开发工作都在云端完成（包括代码、工具和 Antigravity 2.0，简称 AGY）。

### 什么是 Antigravity？
Antigravity 2.0 是 Google 推出的一款独立应用，您可以在其中与能够运行命令、读写文件、搜索网页并使用工具的 AI 智能体协作。在本实验中，您将使用它来构建、测试和部署您的智能体。

> **注意区分智能体！** 在本工作坊中，我们将借助另一个智能体（即编码智能体 Antigravity / AGY）来**构建**一个智能体。在实验过程中请牢记这一区别。

### 打开 Google Cloud 控制台与远程桌面
1. 使用您的邮箱在 Explore 页面注册，展开 **Credentials Reference** 获取临时用户名、密码和项目 ID。
2. 右键点击 **Antigravity in a VM** 并选择“在无痕窗口中打开链接”。
3. 看到剪贴板权限弹窗时选择“Accept”，以便将提示词复制粘贴到 Antigravity 中（注意不要点击右上角关闭按钮）。
4. Antigravity 2.0 已处于打开等待状态，开始前请确保开启一个新对话。
5. **重要提示**：测试本地版 Pitch Generator 时，必须在**远程桌面内的 Chrome 浏览器**中打开；测试部署到 Google Cloud 的版本时，可在远程桌面或本地电脑上访问公共 URL。

### 与 Antigravity 的结对编程工作流
在本实验中，您担任首席软件架构师，Antigravity 担任您的智能结对程序员：
- **以清晰意图提示**：提供简洁、结构化的提示词，明确“做什么（what）”和“为什么（why）”。
- **理解“Why, Prompt, How”循环**：每一步都包含设计原理（Why）、给 Antigravity 的提示词（Prompt）以及实现方式解析（How）。
- **审查契约而非逐字匹配**：由于大语言模型具有非确定性，局部命名或格式可能略有差异，审查时请重点验证核心接口契约（目标文件、类与函数名、状态键）。
- **提示词驱动的验证**：无需手动在终端运行测试命令，可以直接在聊天中要求 Antigravity 验证每一步。

### Lab Helper（实验助手）
工作区在 `.agents/skills/lab-helper/SKILL.md` 中内置了渐进式实验助手技能（以及诊断工具 `verify_workspace.py`）。它仅作为遇到卡顿或测试失败时的“逃生舱”，支持三级渐进提示（Tier 1 概念提示、Tier 2 目标文件与符号提示、Tier 3 实现蓝图）以及工作区修复（Workspace Remediation）。

---

## 检查并运行基线初始应用

在此任务中，您将检查基线四智能体初始应用，在命令行运行它，并将其部署到 Cloud Run。
1. **探索初始代码库**：在 Antigravity 聊天面板中要求 AI 助手生成架构概览并讲解现有代码。
2. **本地运行基线 Pitch Generator**：要求 AI 助手使用测试输入运行智能体并展示提案结果及生成的图像。
   - **注意缺失的功能**：尽管初始应用运行四智能体并行 DAG，具备 `LoopGuard` 循环保护并将图像持久化到 Cloud Storage，但它仍缺乏可复用的品牌指南（`SKILL.md`）、自动化技能合规评估、远程 A2A 微服务隔离、BigQuery 多模态分析与漂移评分、输入安全钩子、PII 脱敏、HITL 审批关卡以及 Token 成本控制。
3. **启动本地服务器并测试**：在远程桌面的 Chrome 中访问 `http://localhost:8080`，输入活动简报并点击生成。如需停止本地服务器，可在运行任务上点击“Stop running task”。
4. **小技巧**：如果对 AI 生成的结果不满意，可以点击提示词容器中的 **Undo changes** 按钮回退对话并修改提示词后重新提交。

---

## 模块 1：扩展智能体团队（Module 1: Expand the agent team）

在此任务中，您将增强四智能体初始工作流（`creative_director`、`brand_strategist`、`copywriter`、`visual_director`），为 Visual Director 装备按需加载的品牌技能，使用确定性测试套件评估合规性，并通过 **Agent2Agent (A2A)** 协议将图像生成解耦为独立微服务。

### 步骤 1a：编写并挂载品牌指南技能
将数百行品牌规则直接嵌入智能体系统提示词会在每轮对话中消耗大量输入 Token。**Agent Skill** 将领域指令存储在带有结构化 YAML frontmatter 的外部 `SKILL.md` 文件中。通过 ADK `SkillToolset` 挂载技能后，智能体在启动时仅看到轻量级元数据，并在需要时通过 `load_skill("brand-guidelines")` 按需加载完整 Markdown 正文。
- **实现方式**：创建包含 YAML frontmatter 和五个必需章节（`# Palette`、`# Light`、`# Composition`、`# Subject`、`# Never`）的 `SKILL.md`；实现 `load_skill_from_dir()` 并挂载到 `visual_director`。
- **压力测试**：使用包含违禁词（如霓虹激光、3D 镀铬标志）的输入测试 Visual Director，验证 `brand-guidelines` 技能是否能将艺术指导拉回品牌内部风格。

### 步骤 1b：评估技能激活与品牌合规性
只有当智能体可靠地调用技能并遵守规则时，技能才真正有效。自动化技能评估框架根据确定性启发式规则对模型输出进行评分：验证是否加载了 `"brand-guidelines"`，检查四个正向品牌支柱（Palette、Light、Composition、Subject），并对违禁词（如 `neon`、`watermark` 等）进行扣分。
- **实现方式**：在单元测试中无需调用在线 LLM 裁判即可进行确定性评分——先检查工具调用历史（`loaded_skills`），再计算四个视觉支柱得分（每项 0.25）并扫描违禁词，仅当技能已加载、无违规且得分 >= 0.75 时判定通过。
- **对比测试**：分别运行合规与明显违规的两个测试用例，比较它们的得分与违规项。

### 步骤 1c：连接远程 A2A Visual Director 服务
高分辨率图像合成计算开销大，适合独立扩展。使用 **Agent2Agent (A2A)** 协议将图像生成解耦为独立微服务，使协调器工作流能够通过 HTTP（`/.well-known/agent-card.json`）调用远程 Visual Director，在使用 `https://` 时自动附加 Cloud Run OIDC Bearer Token，并过滤掉原始的中间 `function_call` JSON 块，同时保留渲染后的 PNG 图像产物。
- **实现方式**：在 `pitch_generator/fast_api_app.py` 中暴露 `build_a2a_visual_director_app()`；在 `pitch_generator/agent.py` 中创建 `_cloud_run_client(url)`（本地 `http://` 免认证，生产 `https://` 附加 OIDC Token 与 600 秒超时）；实现 `_pitch_parts_only(part)` 流过滤器；将本地 `visual_director` 节点替换为 `remote_visual_director`。
- **本地与生产测试**：先在本地同时启动两个微服务并在远程桌面 Chrome 中验证 Agent Card 与前端页面；随后停止本地服务，将两个微服务部署到 Google Cloud Run（约需 5 分钟），并在无痕窗口中验证公共 URL 的端到端 A2A 调用。

---

## 模块 2：观测并审计智能体行为（Module 2: Observe and audit agent behavior）

在此任务中，您将添加企业级可观测性以跟踪智能体遥测数据，将持久化在 Cloud Storage（`GcsArtifactService`）中的主视觉图关联到 **BigQuery** 对象表，并使用多模态 SQL 函数在 BigQuery 中审计品牌合规性。

### 步骤 2a：在 BigQuery 中启用智能体分析
将图像直接存入数据库会导致存储膨胀和查询缓慢。BigQuery **对象表（Object Tables）** 通过维护指向 Cloud Storage 中非结构化文件的元数据引用解决了这一问题。借助 Cloud Resource 连接（`pitch-connection`）上的 `OBJ.MAKE_REF` 和 `OBJ.FETCH_METADATA`，分析师无需复制数据即可用标准 SQL 查询图像文件。
- **实现方式**：预置 BigQuery 数据集（`bwg`）、Cloud Resource 连接（`pitch-connection`）及 IAM 权限（`roles/storage.objectViewer` 和 `roles/aiplatform.user`）；在 `create_key_visuals.sql` 中构建包含 `OBJ.MAKE_REF` 和 `OBJ.FETCH_METADATA` 的 DDL；在 `BigQueryAnalyticsService` 中实现 `record_telemetry` 和 `register_key_visuals`（校验 `gs://` URI）。
- **验证**：运行带实时图像生成的测试并将遥测数据流式传输到 BigQuery，随后在 BigQuery Studio 控制台中运行生成的 SQL 查询检查实时记录。

### 步骤 2b：在 BigQuery 中识别品牌漂移并调优提示词
随着时间推移，视觉模型可能会产生微妙的幻觉或忽略负面约束。BigQuery 的 `AI.SCORE` SQL 函数可直接在 BigQuery 中调用多模态 Gemini 模型，根据品牌评分标准评估生成的视觉效果。得分低于 `7.0` 的视觉图会被标记为 `'needs another pass'`，从而为自动化提示词和技能调优提供闭环触发器。
- **实现方式**：在 `score_brand_fit.sql` 中查询 `bwg.key_visuals` 并调用 `AI.SCORE` 计算 `brand_fit` 与 `verdict`；在 Python 中实现 `detect_brand_drift`（过滤 `brand_fit < 7.0`）和 `tune_prompt_and_skill`（剥离违禁词并强化正向品牌支柱指令）。
- **验证**：使用包含违禁词的漂移提示词测试闭环漂移修复工作流，观察初始评分、调优后的提示词以及重新评分后的 `"on brand"` 结果，并在 BigQuery Studio 中查询 `key_visuals` 表。

---

## 模块 3：加固并保护流水线（Module 3: Harden and secure the pipeline）

在此任务中，您将实施纵深防御安全策略，以防范提示词注入、防止数据泄露，并确保关键营销活动需要人工签核。

### 步骤 3a：通过 PreToolUse 生命周期钩子强制执行工具策略
如果没有授权检查，对抗性简报可能会诱导 LLM 调用危险工具或传入畸形参数（如路径遍历或 Shell 注入命令）。`PreToolUse` 生命周期钩子在每次工具调用执行**前**进行拦截，强制执行严格的工具白名单并清理所有参数。
- **实现方式**：构建默认拒绝（fail-closed）的拦截器，校验 `tool_name` 是否在白名单（`{"load_skill", "generate_key_visual"}`）内，检查 `tool_args` 中是否存在越权参数、路径遍历（`..`）、超长字符串（> 4096 字符）以及提示词注入特征，并将每次检查记录到 `self.audit_log`。
- **红队测试**：使用未授权工具调用、提示词注入攻击以及合法调用三种测试用例对钩子进行测试，检查拦截决策与审计日志。

### 步骤 3b：对输入和输出中的敏感 PII 数据进行脱敏
营销简报经常包含客户联系方式、电话号码或企业邮箱。将原始个人身份信息（PII）发送给云端模型或存入分析日志会违反隐私合规要求。自动化 PII 脱敏会在处理前清理文本和嵌套字典载荷。
- **实现方式**：按优先级顺序应用 `PII_PATTERNS`（先匹配密钥和邮箱，再匹配 16 位信用卡号、SSN 和电话号码），避免误伤正常日期或百分比；通过 `scrub_payload(payload)` 递归克隆字典和列表，在不修改原始输入对象的前提下返回包含 `redaction_count` 的 `ScrubResult`。
- **验证**：使用包含真实发布日期、转化率指标以及多种意外 PII 的混合简报进行测试，确认敏感信息被脱敏且正常日期和指标得以保留。

### 步骤 3c：添加人工审批（HITL）关卡
企业级提案活动绝不应在没有人工监督的情况下自动发布。ADK 的 **Human-in-the-Loop (HITL)** 能力允许工作流在概念构思后暂停执行，发出 `RequestInput` 事件并等待人工审查。如果批准，则继续执行下游文案撰写和图像生成；如果拒绝，工作流立即中止，避免浪费下游 Token。
- **实现方式**：采用双节点 ADK 检查点模式——使用 `@node(rerun_on_resume=False)` 装饰 `approve_concept`，使用 `@node(rerun_on_resume=True)` 装饰 `user_approval`，强制执行严格校验（仅 `"yes"` 或 `"y"` 放行，否则抛出异常短路下游节点）。
- **验证**：启动本地服务器并在浏览器中勾选“Human in the Loop”验证框，分别测试拒绝和批准概念的流程。

---

## 模块 4：通过 Token 经济学优化规模化性能（Module 4: Optimize for scale with tokenomics）

在此任务中，您将使用记忆压缩、滑动窗口历史裁剪和 SHA-256 提示词缓存来优化应用的 Token 经济学，从而控制推理延迟和成本。

### 步骤 4a：实现包含压缩、裁剪与缓存的 Token 经济学管理
长多轮智能体会话会积累冗余的对话历史，导致提示词 Token 呈二次方增长和高延迟。Token 经济学工程应用了三项核心技术：
1. **历史裁剪（History pruning）**：在严格 Token 预算内保留最近几轮的滑动窗口。
2. **记忆压缩（Memory compression）**：将较早的历史上下文总结为单条压缩的系统条目，同时原样保留最近几轮对话。
3. **提示词缓存（Prompt caching）**：使用 SHA-256 对静态领域指令（如 `brand-guidelines`）进行哈希，以在请求间复用缓存前缀。
- **实现方式**：实现 `compress_memory(turns, keep_recent=2)`、`prune_history(turns, max_turns=6, max_tokens=512)` 以及 `PromptCacheManager.get_or_create`（基于 SHA-256 哈希与 TTL 命中/未命中统计），并结合模型层级选择器。
- **基准测试**：在 10 轮头脑风暴会话上对 Tokenomics 管理器进行基准测试，对比压缩和裁剪前后的 Token 数量，并演示连续两次缓存品牌指南前缀时的缓存未命中与命中效果。

---

## 恭喜完成！

恭喜您完成了 **Agent Masterclass**！从基础的多智能体原型出发，您使用 Antigravity 作为 AI 结对程序员，在 Gemini Enterprise Agent Platform 上架构、加固并优化了一个生产级多智能体应用。

### 您学到的内容
- 如何使用自然语言提示词与 Antigravity 高效结对编程。
- 如何为专家智能体装备渐进式按需技能（`SKILL.md`），并使用确定性测试框架评估合规性。
- 如何使用 Agent2Agent (A2A) 协议将繁重工作负载解耦为远程微服务。
- 如何使用 Cloud Storage 产物与 BigQuery `OBJ.MAKE_REF`、`AI.SCORE` 实现多模态可观测性。
- 如何通过 `PreToolUse` 生命周期钩子、PII 脱敏和人工审批关卡实施纵深防御安全。
- 如何利用 Token 经济学（记忆压缩、历史裁剪与提示词缓存）优化推理成本与延迟。

完成工作坊后，您可以前往领取官方的 **Build with Gemini** Google 开发者资料徽章。
