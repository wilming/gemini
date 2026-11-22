# Gemini World · Track 3 — 实验指南（构建智能体优先应用）

在 Google Cloud 上原型设计、装备、评估并部署您自己的智能体优先（Agent-first）应用。

---

## 简介

在本实验中，您将学习如何使用 Google Cloud 工具构建一个完整的、可用于生产的智能体应用。您将完成以下内容：
- 使用 `agents-cli` 原型设计智能体。
- 为智能体配备企业级工具套件，包括记忆（Memory）、会话（Sessions）、工具（Tools）、持久化存储和代码执行。
- 评估并提升智能体响应的质量。
- 将智能体部署到 Agent Platform。
- 赋予智能体使用 Google 生成式模型的能力。
- 构建一个托管在 Cloud Run 上、由 **A2UI** 驱动的智能体优先前端。

---

## 环境准备

在本实验中，您将获得一个带有临时凭证的临时 GCP 环境：
1. 使用您的邮箱在 Explore 页面注册，展开 **Credentials Reference** 获取临时用户名、密码和项目 ID。
2. 右键点击 **Lab Instructions** 并选择“在无痕窗口中打开链接”，使用提供的凭证登录 GCP 控制台。
3. **启动 Workstation**：在顶部搜索栏搜索“Workstation”，启动预置的工作站，并点击右上角按钮在 Code OSS 中打开终端。
   - **重要提示**：请务必在 Workstation 内的 Code OSS 中进行所有开发；如果在 Workstation 外部使用终端，资源将很快耗尽。
4. **身份认证**：在终端中使用提供的凭证运行 `gcloud auth login --update-adc` 完成认证（切勿使用个人凭证）。

### 安装 Skills 与 MCP
在本实验中，我们将使用基于命令行的智能体开发环境 **Antigravity CLI（简称 AGY）** 生成代码：
1. 克隆包含预设技能和 MCP 配置的初始仓库（`build-with-gemini`），其中 `.agents` 目录包含了本实验所需的技能与 MCP 配置。
   - **什么是 Skill？** 一组教导编码智能体如何高效完成特定任务的指令包，在相关时自动加载，无需每次手动拼写所有步骤。
   - **什么是 MCP？** 模型上下文协议（Model Context Protocol），是将外部工具和数据源接入编码智能体的标准方式（例如搜索 Google 官方文档或操作数据库）。
2. 运行 `agents-cli setup --skip-auth` 安装通用智能体开发工具包（基于 Agent Development Kit / ADK，用于脚手架搭建、部署、评估和发布）。
3. 在工作站中点击“Open Folder”打开克隆的 `build-with-gemini` 仓库。

### 启动 Antigravity
1. 运行 `agy` 启动 Antigravity，选择选项 2 使用 Google Cloud 项目和提供的凭证完成登录。
2. **避免智能体混淆**：在本工作坊中，您将借助编码智能体 Antigravity（AGY）来构建您自己的智能体。
3. 使用 `/skills` 和 `/mcp` 命令检查配置是否生效。预期至少应看到两个 MCP：
   - **Developer Knowledge MCP**：为 AGY 提供最新的 Google 官方文档知识（Google Cloud、Firebase、ADK、Agent Platform），避免凭空猜测命令。
   - **Firebase MCP**：让 AGY 能够直接操作 Firestore 等 Firebase 服务。
4. 运行环境验证，调用 `troubleshoot-lab-setup` 技能确认一切配置就绪。

---

## 构建您的第一个智能体

1. **创建基础智能体**：让 AGY 使用 `agents-cli` 构建一个可在本地测试和运行的简单智能体。`agents-cli` 会在后台为您生成完整的 ADK 项目脚手架（Python 智能体定义、配置及本地运行环境），核心逻辑位于 `app/agent.py`。
   - **小贴士**：如果退出终端，之后重新打开 AGY 并运行 `/resume` 即可恢复历史会话。
2. **使用 Playground**：Playground 是本地基于浏览器的聊天界面，除了聊天外，您还可以在左侧查看智能体拥有的工具（如 `get_weather`），在 **Traces** 标签页查看每一步的调用链路，并在顶部栏管理 **Sessions**（会话）和临时状态。
3. **部署到 Agent Platform**：通过 AGY 将智能体部署到 Google Cloud Agent Platform，并在控制台中查看生产环境监控工具套件。

---

## 设计您的应用

接下来，您将把基础聊天机器人演进为您自己设计的智能体应用，增加工具调用、跨会话记忆、数据库/对象存储访问、图像生成和前端界面。

优秀的实验项目通常聚焦于一个带有少量数据的明确任务，例如：
- **食谱助手**：根据现有食材推荐菜谱并保存收藏。
- **图书顾问**：推荐书单、追踪阅读进度并生成封面艺术图。
- **健身教练**：制定训练计划、记录训练并绘制进度图表。
- **旅行规划师**：编排行程并为每个目的地绘制插图。
- **植物护理助手**：追踪植物状态、提醒浇水并根据照片诊断问题。

### 生成项目简报与重命名
1. 调用 `pick-your-agent-project` 技能，与 AGY 头脑风暴并生成 `project_brief.md` 作为后续开发参考。
2. 根据 `project_brief.md` 重命名现有智能体项目文件夹以及 `agents-cli-manifest.yaml` 和 `pyproject.toml` 中的名称。

---

## 添加记忆（Add Memory）

`Sessions` 仅存储当前对话的信息，而 **Memory Bank** 能够实现跨会话记忆（例如记住客户讨厌红色）。每次用户发送消息时，Memory Bank 会自动分析对话并提取有价值的事实片段供未来会话使用。

> **重要提示**：Memory Bank 仅在**已部署**到 Agent Platform 的智能体上生效（记忆在服务端提取），本地 Playground 不会提取记忆。测试前请确保已重新部署并在浏览器中与已部署的智能体对话。

使用 `memory-bank-setup` 技能为智能体配置 Vertex AI Memory Bank 实例并重新部署，随后在 Agent Platform UI 中查看已提取的记忆。

---

## 添加持久化存储（Add Persistent Storage）

Memory Bank 适合记录用户的对话偏好，但应用本身的数据（如商品目录、预订记录、库存或生成的图像）需要专用的持久化存储：
- **结构化数据** → **Firestore**（无服务器、简单易用）。
- **原始文件/二进制大对象** → **Cloud Storage (GCS)**。

### 配置 Firestore 后端
借助 Firebase MCP，让 AGY 根据 `project_brief.md` 为您的应用创建 Firestore 集合、读写工具函数及初始种子数据。
- **注意**：需将项目 ID 硬编码为字符串，不要从 `google.auth.default()` 或 `GOOGLE_CLOUD_PROJECT` 读取（部署到 Agent Platform 后它们会返回项目编号，导致 Firestore 报错）。
- Firestore 具有实时性，您在控制台中的修改会立即反映到智能体中，反之亦然。

### 在 Cloud Storage 中存储文件
创建一个公开可读的 Cloud Storage 存储桶，用于存放智能体生成的媒体文件（如图像、音视频），并返回公开的 `https` URL 以便在卡片和自定义前端中直接展示。

---

## 添加工具（Add Tools）

工具是智能体用来执行实际操作的函数（如查询数据库、发送邮件等），这标志着智能体从“聊天机器人”转变为“行动者”。根据 `project_brief.md` 让 AGY 建议 2–3 个工具，选择最简单的一个实现为函数工具并在 Playground 中测试其调用链路。

---

## 使用 RAG 为智能体提供知识依据（Ground Your Agent with RAG）

当参考文档（如产品手册、政策汇编）过大而无法直接塞入提示词时，**RAG（检索增强生成）** 将文档分块、向量化并索引到托管向量数据库（Corpus）中，智能体在运行时通过检索工具语义搜索最相关的片段并据此作答。

使用 `rag-engine-setup` 技能创建无服务器（Serverless）Vertex AI RAG 语料库，导入并索引参考文档（如 `.txt` 文件），并添加检索工具，随后在 Playground 中验证智能体是否基于检索片段作答。

---

## 生成图像（Generate Images）

使用 Google 最快的图像模型 **`gemini-3.1-flash-lite-image`（Nano Banana 2 Lite，运行在 `global` 区域）** 为智能体添加按需生图工具。该工具对生成的图像执行两项操作：
1. 使用 `tool_context.save_artifact` 保存，以便在 Playground 的 Artifacts 面板中快速预览。
2. 上传图像字节到此前创建的公开 Cloud Storage 存储桶，并返回公开的 `https` URL，以便在 A2UI 卡片和自定义前端中加载显示。

---

## 在沙箱中运行代码（Run Code in a Sandbox）

当智能体需要进行快速 API 调用、数据分析或数学计算时，Agent Platform 提供了隔离的沙箱代码执行环境（`AgentEngineSandboxCodeExecutor`）。状态跨会话持久化且不触及宿主机，非常适合在企业环境中安全运行模型生成的代码。
- **前提**：已启用 Agent Platform API，且智能体身份拥有 `roles/aiplatform.user` 权限。

---

## 使用 A2UI 丰富响应界面（Enrich Responses with A2UI）

**A2UI** 让智能体能够以真实的 UI 组件（卡片和表格，而非纯文本）进行回复。智能体输出描述 UI 的 JSON，渲染器将其转化为界面组件。
使用 `enable-a2ui` 技能完成两项配置：
1. 使用 `A2uiSchemaManager`（v0.8 版本和 Basic Catalog）构建系统提示词，教导模型输出有效的 A2UI JSON。
2. 将 `a2ui_utils.py` 作为 `after_model_callback` 接入智能体，将模型 JSON 包装为 ADK 开发 UI 渲染器所需的格式。

### 部署前在本地验证 A2UI
在本地重启 Playground 进行测试。注意以下关键点：
- **必须关闭 Playground 设置中的 Token Streaming**，否则开发 UI 会直接显示原始流式 JSON 而不会渲染卡片。
- 若要在卡片中显示图像，需使用 `Image` 组件并填入 Cloud Storage 的公开 `https` 链接（纯产物文件名无法被渲染器加载）。

---

## 构建前端（Build a Frontend）

现在为您的智能体打造一个可分享的 Web 界面并部署到云端：
1. **重新部署完整的智能体**：将包含存储、工具、媒体、沙箱和 A2UI 的最新版本重新部署到 Agent Platform。
2. **授予运行时权限**：使用 `troubleshoot-lab-setup` 技能为已部署智能体的服务账号授予 `roles/datastore.user`（访问 Firestore）和 `roles/storage.objectAdmin`（上传图像到存储桶）。
3. **构建最小化前端**：使用 `build-agent-frontend` 技能将包含 FastAPI 代理和聊天 UI 的模板复制到 `frontend/` 目录，并绑定 `AGENT_ENGINE_RESOURCE_NAME` 和 `AGENT_DIRECTORY`。浏览器仅与同源代理通信（无 CORS 问题），代理使用默认应用凭证与已部署智能体认证通信。
4. **本地测试**：在 `frontend/` 目录下安装依赖并启动本地服务（`http://localhost:8080`），验证对话与会话记忆是否正常连通远程智能体。
5. **部署到 Cloud Run**：将独立的前端 Web 应用通过 `gcloud run deploy` 部署到 Cloud Run，并为 Cloud Run 服务账号授予 `roles/aiplatform.user` 权限。至此，智能体运行在 Agent Platform，前端运行在 Cloud Run，形成完整闭环。

---

## 自定义前端（Customize Your Frontend）

整个前端 UI 位于 `frontend/static/index.html` 单文件中。您可以让 AGY 调整标题、页眉、主题强调色，在输入框上方添加 3 个可点击的示例提示词，或优化对话布局。

---

## 进阶目标（Stretch Goals）

完成基础闭环后，您可以进一步尝试：
- **使用 Omni 生成视频预览**：调用 `gemini-omni-flash-preview`（`global` 区域）根据文本或参考图像生成短视频，同样保存为 Artifact 并上传至 Cloud Storage 返回公开 URL。
- **查看 Cloud Trace**：查看已部署智能体导出的完整执行链路（每次 LLM 调用与工具调用）。
- **持续定制前端品牌风格**。

---

## 分享您的成果（Share What You Built）

### 录制演示视频
使用仓库内置的 `record-demo` 技能，在无头浏览器中自动打开聊天 UI、输入 2–3 个展示核心功能（工具调用、Firestore 查询、生图、A2UI 卡片）的提示词，并录制一段带有“Gemini World Tour”边框的 `.webm` 短视频（约 20–40 秒）。还可选用 Google 的 Lyria 模型为其配乐。

### 生成可分享的 README
让 AGY 为项目生成 `README.md`，介绍智能体功能与所使用的 Google Cloud 工具，并将录制的 `.webm` 视频转换为循环播放的优化 GIF 嵌入顶部（使用相对路径即可在 GitHub 上自动循环播放）。

---

## 发布到 GitHub 并提交领取纪念品

由于实验工作站是临时的，实验结束后代码将被清除。最后一步是使用 `publish-to-github` 技能将项目推送到您**个人**的 GitHub 账号：
1. 通过 GitHub CLI 的一次性设备码在个人设备上登录您自己的 GitHub 账号（而非 Qwiklabs 账号）。
2. 将仓库命名为 `buildwithgemini-[YOUR-APP-NAME]` 并设为公开（Public），AGY 会在提交前自动生成 `.gitignore` 并扫描密钥。
3. 完成后打开预填充的提交表单，填写您的注册信息即可参与领取纪念品（卫衣）并兑换 Google 开发者徽章（GDP Badge），优秀项目还有机会入选 Build with Gemini Track 3 GitHub 展示画廊。

恭喜您设计、构建、装备、评估、部署、分享并发布了一个完整的智能体应用！
