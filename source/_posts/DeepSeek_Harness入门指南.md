---
title: DeepSeek Harness 入门指南：安装、四种模式、插件与模型切换
date: 2026-08-16 12:00:00
tags: [DeepSeek Harness, DSH, Agent, 插件生态]
---

> 适用环境：Windows / DeepSeek Harness 0.1.0-rc.6

DeepSeek Harness，简称 DSH，是 DeepSeek 推出的开源 Agent 框架。

它不只是给模型套了一个聊天界面。模型可以在里面读取和编辑文件、执行终端命令、搜索代码、调用插件，还能根据任务切换不同的 Agent 工作方式。

这篇文章分成四部分：

1. 安装流程
2. DSH 四种模式介绍
3. 插件推荐，以 dshmarket 为核心
4. DeepSeek 模型切换

目前 DSH 仍处于开发预览阶段，版本更新可能改变配置结构和插件接口。遇到界面或命令不一致时，以 [DeepSeek Harness 官方仓库](https://github.com/deepseek-ai/deepseek-harness) 的最新说明为准。

<!-- more -->

## 01｜安装流程

### 安装 Node.js

先安装 Node.js，建议选择当前 LTS 版本。

安装完成后，重新打开 PowerShell，检查 Node.js 和 npm：

```powershell
node --version
npm --version
```

DSH 的部分插件会使用 pnpm 安装依赖，建议一起安装：

```powershell
npm install -g pnpm
pnpm --version
```

如果终端提示：

```text
'pnpm' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```

一般有两个原因：pnpm 尚未安装，或者安装后的 PATH 还没有在当前终端中刷新。执行上面的安装命令，然后关闭并重新打开终端即可。

### 安装 DeepSeek Harness

如果只是想临时体验，可以直接运行官方提供的命令：

```powershell
npx @deepseek-ai/dsh web
```

如果准备长期使用，建议全局安装：

```powershell
npm install -g @deepseek-ai/dsh
dsh --version
dsh web
```

看到服务启动成功后，在浏览器打开：

```text
http://127.0.0.1:3080
```

这里有一个容易误解的地方：`dsh web` 默认是前台进程，不是自动注册的 Windows 常驻服务。

终端关闭、进程退出或者电脑重启以后，Web 服务都会停止。下次使用时，需要重新执行：

```powershell
dsh web
```

因此，运行后终端一直“卡住”并不是故障，而是 DSH 正在占用当前终端提供 Web 服务。可以另外打开一个终端处理其他事情。

如果希望开机自动运行，可以后续使用 Windows 任务计划程序、PM2 或社区启动器。不过刚开始使用时，建议先保留前台运行，这样启动失败时可以直接看到完整日志。

![alt text](/image/deepseek1.jpg)

## 02｜DSH 四种模式介绍

DSH 中的“模式”并不是四个不同模型，也不是简单的权限等级，而是四套 Agent 预设。它们决定模型能看到哪些运行时信息、能够使用哪些工具，以及如何组织工具调用。

### 标准模式：日常使用首选

标准模式提供完整的编码 Agent 能力，包括：

- 读取和编辑文件
- 执行 Shell 命令
- 搜索本地文件和网页
- 使用 Skills
- 制订计划和目标
- 调用子 Agent 与工作流

写代码、修改项目、排查报错、阅读仓库时，都可以优先选择标准模式。如果不知道选什么，就从标准模式开始。

### PTC 模式：把多步工具调用组合起来

PTC 模式具备标准模式的大部分能力，但工具调用方式不同。

标准模式通常是模型调用一次工具、读取结果，再决定下一步。PTC 模式则允许模型生成一段 TypeScript 程序，通过 Code Mode SDK 把多项操作组合到一次执行中。

例如，一个任务需要读取五个文件、搜索三个关键词，再汇总结果。标准模式可能需要多轮工具调用；PTC 模式可以把这些步骤写进一个程序中完成。

它适合批量读取、并行搜索和多步骤数据整理。代价是执行逻辑更复杂，出现问题时不如标准模式直观。

### 极简模式：只保留两个基础工具

极简模式只提供持久 Bash 和文本编辑器两类工具，同时减少运行时上下文与自动化组件。

它适合：

- 小范围代码修改
- 测试基础工具链
- 希望降低上下文开销
- 观察模型如何使用最基础的编码工具

面对大型项目和复杂任务时，极简模式通常没有标准模式方便。

### 创造模式：为开发自定义 Agent 准备

创造模式主要面向 DSH 高级用户和插件开发者。

除了标准模式的能力，它还可以检查 DSH 运行时、实验插件，并创建新的 Agent preset。也就是说，用户可以在 DSH 里面继续改造 DSH。

创造模式能够对实时运行环境执行模型生成的代码，应该把它视为完整的本机 Shell 访问能力。不要在不可信项目或包含敏感凭据的环境中随意使用。

简单总结：

```text
日常编程：标准模式
批量、多步骤操作：PTC 模式
简单修改和工具实验：极简模式
插件与 Agent 开发：创造模式
```

## 03｜插件推荐：先装 dshmarket

DSH 很重要的一项设计就是插件化。界面组件、工具、命令和 Agent 能力，都可以通过插件扩展。

普通用户最值得优先安装的插件是 `dshmarket`：

```powershell
dsh plugin --profile web add dshmarket
```

安装完成后，停止当前的 DSH 进程，然后重新启动：

```powershell
dsh web
```

进入设置页面后，就可以通过 Plugin Market 浏览、搜索、安装、更新和卸载社区插件。

`dshmarket` 相当于 DSH 的插件市场入口。除了插件搜索和管理，它还提供备份、恢复、来源限制等功能，并默认限制部分依赖安装脚本，以降低供应链风险。详细说明可以查看 [dshmarket 项目仓库](https://github.com/dsh-market/dsh-market)。

有了 dshmarket，再根据自己的需求选择插件。下面几款比较实用。

![alt text](/image/deepseek2.png)

### dsh-context：查看上下文是怎么被占满的

```powershell
dsh plugin --profile web add dsh-context
```

它可以观察系统提示词、项目文件和工具结果分别占用了多少上下文。适合排查对话为什么越来越慢，或者模型为什么开始遗忘前面的内容。

项目地址：[dsh-context](https://github.com/bowenliang123/dsh-context)

![alt text](/image/deepseek3.png)

### deepseek-harness-zh_pro：中文界面

```powershell
dsh plugin --profile web add deepseek-harness-zh_pro
```

它为 DSH 提供更完整的中文界面，适合希望将菜单、设置和提示信息中文化的用户。

项目地址：[deepseek-harness-zh_pro](https://github.com/magian1127/deepseek-harness-zh_pro)

### dsh-doctor：检查配置和运行环境

```powershell
dsh plugin --profile web add dsh-doctor
```

它主要用于检查 DSH 配置、插件状态和常见环境问题，适合定位启动失败或配置冲突。

项目地址：[dsh-doctor](https://github.com/asdf17128/dsh-doctor)

更多插件可以在 [Awesome DSH Plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin) 中查找。

不过需要提醒一句：进入社区列表不等于通过安全审计。DSH 插件通常以当前 Windows 用户的权限运行，理论上可以读取文件、访问网络和执行命令。安装前最好确认源码公开、维护仍然活跃，并检查插件要求的权限和安装脚本。

## 04｜模型切换

这里需要先纠正一个常见误解：DeepSeek Harness 并不只能使用 DeepSeek 模型。

DSH 有两层模型接入方式：

1. 官方的 DeepSeek 直连适配器，默认提供 DeepSeek-V4-Flash 和 DeepSeek-V4-Pro。
2. 基于 `pi-ai` 的多提供商适配器，可以接入 OpenAI、Anthropic、Google、OpenRouter 等大量平台。

官方仓库将这两层分别称为 `llm-deepseek` 和 `llm-pi-ai`。后者不仅能读取内置模型目录，还允许添加 OpenAI-compatible 网关、自托管服务以及目录中尚未收录的新提供商。相关设计可以查看 [DSH 的 LLM 适配器说明](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/llm) 和 [`llm-pi-ai` 配置文档](https://github.com/deepseek-ai/deepseek-harness/tree/master/packages/llm/llm-pi-ai)。

按本文使用的 `dsh 0.1.0-rc.6` 统计，它内置的 `pi-ai 0.82.1` 模型目录包含：

```text
37 条提供商路由
1,109 条模型目录记录
```

这里的“1,109 条”不等于 1,109 个完全不同的模型。OpenRouter、Vercel AI Gateway、Amazon Bedrock 等平台可能同时收录相同或相近的底层模型；中国区、国际区、Token Plan 和 Coding Plan 也可能被拆成不同路由。因此，更准确的说法是：DSH 当前内置了 30 多条提供商路由，目录中有上千条可配置的模型记录。

主要提供商包括：

- DeepSeek、OpenAI、Anthropic、Google、Mistral、xAI
- MiniMax、Moonshot AI、Kimi Coding、Qwen、ZAI、Xiaomi MiMo、蚂蚁 Ling
- Azure OpenAI、Google Vertex AI、Amazon Bedrock、NVIDIA NIM
- OpenRouter、Vercel AI Gateway、Cloudflare AI Gateway、Cloudflare Workers AI
- Hugging Face、Together AI、Fireworks、Groq、Cerebras
- GitHub Copilot、OpenAI Codex、OpenCode Zen、OpenCode Go

另外还有中国区、国际区和不同订阅计划对应的独立路由。`pi-ai` 的官方列表也明确说明，它支持任意 OpenAI-compatible API，包括 Ollama、vLLM 和 LM Studio。完整且持续更新的名单可以查看 [`pi-ai` Supported Providers](https://github.com/earendil-works/pi/tree/main/packages/ai#supported-providers)。

但“出现在提供商目录中”不代表可以直接免费使用。多数平台仍然需要填写对应 API Key；GitHub Copilot、OpenAI Codex 等提供商可能使用 OAuth 或已有订阅；云平台还可能需要项目、区域或部署名称。只有完成凭据配置并启用对应路由后，它的模型才会出现在可用模型列表中。

### 添加模型提供商

进入 DSH 设置中的模型或提供商页面，选择需要的平台，然后填写 API Key 或完成 OAuth 登录。不同版本的中文插件可能会把入口翻译成“模型”“提供商”或“Models”。

常见选择可以这样理解：

- 已有官方 API Key：直接选择 OpenAI、Anthropic、Google、DeepSeek 等提供商。
- 希望一个 Key 使用多家模型：选择 OpenRouter 或其他模型网关。
- 已有 ChatGPT、Copilot 等订阅：选择对应的 OAuth/订阅提供商，具体可用模型取决于账号权限。
- 本地运行 Ollama、vLLM 或 LM Studio：添加 OpenAI-compatible 提供商，填写 Base URL 和模型 ID。

保存配置后，再回到对话页面切换模型。

### 在对话中切换模型

在 Web 界面中，有两个常用的切换入口：

1. 点击输入框附近的模型选择器。
2. 在输入框中执行 `/model`，打开模型菜单。

模型菜单会按照提供商对已配置的模型进行分组，并提供“模型”和“推理强度”两层选择。输入框附近的模型选择器可以同时调整模型与推理强度；使用 `/model` 切换时，通常采用该模型的默认推理强度。具体交互可以参考 [DSH 官方模型选择说明](https://github.com/deepseek-ai/deepseek-harness/blob/master/packages/client/ui-model-selection/README.zh.md)。

### DeepSeek-V4-Flash

Flash 更适合日常高频任务：

- 快速浏览项目
- 普通代码编写
- 简单报错修复
- 高频、多轮交互
- 对响应速度比较敏感的任务

它可以作为默认模型，覆盖大多数日常工作。

### DeepSeek-V4-Pro

Pro 更适合复杂任务：

- 架构设计
- 困难故障排查
- 大范围代码重构
- 深度代码审查
- 需要长链条推理的任务

比较实用的使用方式是：先用 Flash 浏览项目、收集信息和完成普通修改，遇到复杂决策或疑难问题时，再切换到 Pro。

上面的 Flash 和 Pro 只是 DeepSeek 官方直连下的两个选项，不是 DSH 的全部模型。

模型切换会在下一次 Agent 请求边界生效，不会改变已经开始执行的当前步骤。跨提供商切换时，历史对话仍然保留，但不同平台的缓存和推理元数据不一定能够复用。更适合在任务阶段发生变化时切换，而不是每发送一句话就换一次模型。

![alt text](/image/deepseek4.png)

## 最后

如果第一次使用 DSH，我建议从下面这套组合开始：

```text
模式：标准模式
模型：DeepSeek-V4-Flash
插件：dshmarket、dsh-context、dsh-doctor
```

日常工作使用标准模式和 Flash；复杂任务切换到 Pro；需要批量调用工具时尝试 PTC；只有开发插件或自定义 Agent 时，再进入创造模式。

先把核心链路跑通，再逐步添加插件，比一开始装满整个插件市场更稳。
