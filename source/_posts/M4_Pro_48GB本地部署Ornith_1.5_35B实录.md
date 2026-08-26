---
title: 我在 48GB M4 Pro 上跑起 Ornith 1.5 35B：本地生成接近 57 token/s
date: 2026-08-26 12:00:00
tags: [Ornith 1.5, llama.cpp, GGUF, Apple Silicon, 本地部署]
---

> 实测机器：MacBook Pro，Apple M4 Pro，48GB 统一内存
> 运行方案：Ornith-1.5-35B-A3B Q4_K_M + llama.cpp Metal

前面在这台 M4 Pro 上折腾 Qwen3.8-27B，普通 4-bit 生成大约 15 token/s，加上 DFlash2 后能到 27～31.5 token/s。后来看到 Ornith 1.5，我又装了它的 35B-A3B 版本。本来只是想看看这个偏编程和 Agent 的 MoE 模型能不能在 48GB Mac 上稳定运行，结果文本生成直接跑到了 51.5～56.98 token/s，图片理解和 OpenCode 工具调用也都走通了。

这次没有继续折腾 MLX，也没有额外加草稿模型。我选择的是约 21.7GB 的 Q4_K_M GGUF，通过 Homebrew 安装的 llama.cpp 在 Metal 后端运行，视觉部分再加载一个约 0.9GB 的 mmproj。整套部署最后保留文本对话、图片理解、OpenAI 兼容 API 和 OpenCode 四种入口。

这篇只记录我实际安装成功的流程。命令以 `~/Develop/AI` 为示例，换成自己的目录也可以。

<!-- more -->

## 01｜我的机器与最终安装组合

实测机器是 14 核 CPU、20 核 GPU 的 Apple M4 Pro，统一内存 48GB。运行时使用 Homebrew 安装的 llama.cpp，Metal 全层卸载，并开启 Flash Attention。

最终组合如下：

| 组件 | 版本或文件 | 占用 |
|---|---|---:|
| 主模型 | `Ornith-1.5-35B-Q4_K_M.gguf` | 约 21.7GB |
| 视觉投影 | `mmproj-Ornith-1.5-35B-BF16.gguf` | 约 0.9GB |
| 推理框架 | Homebrew 安装的 llama.cpp | 由 Brew 管理 |
| 默认上下文 | 32768 token | 可通过环境变量调整 |
| API 地址 | `http://127.0.0.1:8000/v1` | 仅本机监听 |

`35B-A3B` 是一个容易看错的名字。它不是只有 3B 权重，而是总参数约 35B、每个 token 激活约 3B 参数的 MoE 模型。完整专家权重仍然要装进统一内存，所以磁盘和加载内存要按 35B 模型准备；但实际生成时每一步不用把所有参数都算一遍，这也是它明显快于Qwen3.8-27B 稠密模型的重要原因。


## 02｜用 Homebrew 安装 llama.cpp 和 uv

先建立模型目录：

```bash
mkdir -p ~/Develop/AI/Ornith-1.5-35B-A3B-GGUF
cd ~/Develop/AI/Ornith-1.5-35B-A3B-GGUF
```

`llama.cpp` 和 Python 工具都直接交给 Homebrew 管理：

```bash
brew install llama.cpp uv
```

## 03｜用 uv 下载 Q4_K_M 主模型与视觉投影

我用 `uv` 创建独立 Python 环境并安装 Hugging Face CLI，避免依赖进入系统 Python：

```bash
uv venv --python 3.12
uv pip install --python .venv/bin/python huggingface-hub
mkdir -p model
```

接下来从官方 GGUF 仓库下载主模型和视觉投影：

```bash
./.venv/bin/hf download \
  ornith-ai/Ornith-1.5-35B-A3B-GGUF \
  Ornith-1.5-35B-Q4_K_M.gguf \
  mmproj-Ornith-1.5-35B-BF16.gguf \
  --local-dir ./model
```

主模型约 21.7GB，视觉投影约 0.9GB，再算上下载缓存，建议至少预留 25GB 磁盘。只想跑文本也可以暂时不加载 mmproj，但后面的图片理解和多模态 API 就不能用了。

## 04｜分别跑通文本和图片理解

先不要急着接 API，直接用 `llama-cli` 验证模型最容易定位问题。文本对话命令如下：

```bash
llama-cli \
  --model ./model/Ornith-1.5-35B-Q4_K_M.gguf \
  --ctx-size 32768 \
  --n-gpu-layers 99 \
  --flash-attn on \
  --jinja \
  --reasoning on \
  --reasoning-format deepseek \
  --temperature 0.6 \
  --top-p 0.95 \
  --top-k 20 \
  --multiline-input
```

`--n-gpu-layers 99` 会尽可能把模型层卸载到 Apple GPU；`--jinja` 使用模型携带的聊天模板；`--reasoning-format deepseek` 让思考内容按 llama.cpp 支持的格式解析。进入交互界面后，先问一个短问题，确认中文输出和结束标记都正常。
如果 32K 上下文占用偏高，把两条命令中的 `--ctx-size 32768` 改成 `16384` 即可。

## 05｜启动 OpenAI 兼容 API

CLI 跑通后，再用 `llama-server` 启动本地 API：

```bash
llama-server \
  --model ./model/Ornith-1.5-35B-Q4_K_M.gguf \
  --mmproj ./model/mmproj-Ornith-1.5-35B-BF16.gguf \
  --alias Ornith-1.5-35B-A3B-Q4_K_M \
  --host 127.0.0.1 \
  --port 8000 \
  --ctx-size 32768 \
  --n-gpu-layers 99 \
  --flash-attn on \
  --jinja \
  --reasoning on \
  --reasoning-format deepseek \
  --temperature 0.6 \
  --top-p 0.95 \
  --top-k 20
```

服务监听 `127.0.0.1`。另开一个终端检查模型列表：

```bash
curl http://127.0.0.1:8000/v1/models
```

再发一条 OpenAI 格式的聊天请求：

```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Ornith-1.5-35B-A3B-Q4_K_M",
    "messages": [
      {"role": "user", "content": "用一句话确认本地 API 工作正常。"}
    ],
    "max_tokens": 128,
    "temperature": 0
  }'
```

模型列表和聊天请求都返回成功后，说明从 GGUF、Metal 到 OpenAI 兼容协议的整条链路已经打通。

## 06｜接入 OpenCode，当成本地 Coding Agent

OpenCode 不能直接调用 `llama-cli`，它需要一个兼容 API。把下面的 provider 合并到 OpenCode 配置文件的 `provider` 对象中：

```json
"ornith15": {
  "npm": "@ai-sdk/openai-compatible",
  "name": "Local Ornith 1.5",
  "options": {
    "baseURL": "http://127.0.0.1:8000/v1",
    "apiKey": "local",
    "timeout": 600000
  },
  "models": {
    "ornith15-35b": {
      "id": "Ornith-1.5-35B-A3B-Q4_K_M",
      "name": "Ornith 1.5 35B-A3B Q4_K_M",
      "reasoning": true,
      "temperature": true,
      "tool_call": true,
      "limit": {
        "context": 32768,
        "output": 8192
      },
      "modalities": {
        "input": ["text", "image"],
        "output": ["text"]
      }
    }
  }
}
```

先启动上一节的 `llama-server`，再进入需要处理的项目目录运行：

```bash
opencode --model ornith15/ornith15-35b
```

## 07｜实测 51.5～56.98 token/s，为什么 MoE 这么快

这套配置在 M4 Pro 48GB 上的短文本生成速度稳定在 51.5～56.7 token/s。接入 OpenAI 兼容 API 后，我又测到约 56.98 token/s；图片理解测试约 54.3 token/s。不同提示词、输出长度和是否包含推理内容都会带来波动，所以我更愿意把结果写成一个区间，而不是只保留最好看的峰值。

| 测试入口 | 本机结果 |
|---|---:|
| 文本 CLI | 约 51.5～56.7 tok/s |
| 图片理解 | 约 54.3 tok/s |
| OpenAI 兼容 API | 约 56.98 tok/s |
| 工具调用 | 已正常返回并完成转换 |

这个速度明显高于我之前测试的 Qwen3.8-27B 稠密模型。后者普通解码约 15 token/s，加上 DFlash2 后约 27～31.5 token/s；Ornith 35B-A3B 没有额外草稿模型，也能接近 57 token/s。

关键不在“35B 比 27B 更小”，而在 MoE 的激活方式。Ornith 的完整 35B 权重依然需要驻留内存，但每生成一个 token 只激活其中约 3B 参数。Apple Silicon 上逐 token 生成很容易受内存带宽限制，减少每一步真正参与计算和读取的参数量，就能显著提高解码吞吐。

不过，token/s 不是模型能力分数。MoE 激活参数少，并不意味着它等价于一个普通 3B 模型；同样，生成更快也不代表每个编程任务都一定完成得更好。OpenCode 的端到端速度还会受到上下文预填充、工具执行时间、调用轮数和模型是否走弯路影响。


---

## 参考来源

1. [Ornith-1.5-35B-A3B GGUF](https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B-GGUF)
2. [Ornith-1.5-35B-A3B 模型卡](https://huggingface.co/ornith-ai/Ornith-1.5-35B-A3B)
3. [llama.cpp](https://github.com/ggml-org/llama.cpp)
4. [OpenCode](https://opencode.ai/)
