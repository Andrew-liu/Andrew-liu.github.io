---
title: 我在 32GB M1 Pro 上跑起了 Qwen3.8-27B：实测 8.7 token/s
date: 2026-08-20 12:00:00
tags: [Qwen3.8, 本地部署, M1 Pro, macOS]
---

![alt text](/image/m1_x.png)

昨天看了L总的推，心里直痒痒，27B 参数的模型到底能不能跑在一台 32GB 内存的老款 M1 Pro？

我原本觉得有点悬。毕竟内存不是只给模型用，macOS、Python、推理框架、KV Cache 全都要抢这 32GB。真把模型加载起来以后，结果比我预想的好：4-bit 版本可以稳定运行，本地 API 也能正常对话，生成速度大约 8～8.7 token/s。

不算快，但已经脱离“跑个 Demo 看一眼”的阶段。日常问答、改代码、写点东西，完全能用。

这篇只记录我实际走通的方案：Apple Silicon Mac、MLX-VLM、Qwen3.8-27B 4-bit，以及几个很容易踩错的开关。

<!-- more -->

## 01｜先看机器：32GB 是这套方案的起点

我测试的机器是：

- MacBook Pro 2021
- Apple M1 Pro，10 核 CPU、16 核 GPU
- 32GB 统一内存
- Metal 可用

MLX 的优势就是能直接利用 Apple Silicon 的统一内存。CPU 和 GPU 不需要各存一份模型，27B 的 4-bit 权重才有机会在 32GB 机器上落地。

这里要先把预期说清楚：32GB 能跑，但余量不算奢侈。模型权重本身约 16.05GB，实际推理峰值还会升到 18.5～21.7GB。

如果你只有 16GB，我不建议硬上这个 27B；如果是 24GB，可以试 4-bit，但上下文不要拉太长；32GB 才是相对舒服的起点。

## 02｜给模型单独建目录，环境也隔离开

我没有把模型、Python 包和启动脚本全扔进一个公共的 LLM 目录，而是单独建了一个 `Qwen3.8-27B` 目录。后面升级、测速、删除都不会影响其他模型。

```bash
mkdir -p ~/Develop/LLM/Qwen3.8-27B
cd ~/Develop/LLM/Qwen3.8-27B

uv python install 3.12
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python \
  "mlx-vlm>=0.6.13" \
  "mlx>=0.32" \
  huggingface-hub
```

我这里用 Python 3.12，和系统自带的Python版本隔离。

安装完检查一下：

```bash
./.venv/bin/python -c \
  "import mlx, mlx_vlm; print('mlx:', mlx.__version__, 'mlx-vlm:', mlx_vlm.__version__)"
```

## 03｜只下载这个：4-bit

模型仓库是：

`orcarouter/Qwen3.8-27B-Uncensored-MLX`

![alt text](/image/m1_hg.png)

它提供 2-bit、4-bit、6-bit 和 8-bit。我这台 32GB M1 Pro 选的是 **4-bit**，下载后的权重约 16.05GB。

原因很简单：2-bit 虽然只占约 9.36GB，但模型作者明确提示质量损失严重；6-bit 和 8-bit 分别约 22.8GB、29.5GB，给系统和推理缓存留下的空间太小。32GB 机器上，4-bit 是质量、速度和内存余量之间最实际的平衡点。

这个仓库需要先在 Hugging Face 页面接受访问条件，并完成 CLI 登录：

```bash
./.venv/bin/hf auth login

./.venv/bin/hf download \
  orcarouter/Qwen3.8-27B-Uncensored-MLX \
  --include "4-bit/*" \
  --local-dir ./Qwen3.8-27B-Uncensored-MLX
```

下载完成后，真正的模型目录是：

```text
./Qwen3.8-27B-Uncensored-MLX/4-bit
```

## 04｜先做测试，再一键拉起本地 API

先用模型文档给出的 `mlx_vlm` 入口做最小测试：

```bash
./.venv/bin/python -m mlx_vlm generate \
  --model ./Qwen3.8-27B-Uncensored-MLX/4-bit \
  --prompt "用一句话解释什么是统一内存。" \
  --max-tokens 128
```

能正常输出后，再启动 OpenAI 兼容 API：

```bash
MODEL_PATH="$(pwd)/Qwen3.8-27B-Uncensored-MLX/4-bit"

./.venv/bin/python -m mlx_vlm server \
  --model "$MODEL_PATH" \
  --host 127.0.0.1 \
  --port 8080 \
  --max-tokens 1024
```

这就是模型文档里的启动方式，只额外补了 `--host 127.0.0.1` 和输出长度。我把模型目录先转成绝对路径，因为 API 请求必须使用同一个模型标识。服务只监听本机，不会直接暴露给局域网。

API 地址是：

```text
http://127.0.0.1:8080/v1
```

用下面这条命令测试完整对话链路：

```bash
MODEL_PATH="$(pwd)/Qwen3.8-27B-Uncensored-MLX/4-bit"

curl http://127.0.0.1:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"$MODEL_PATH\",
    \"messages\": [
      {\"role\": \"user\", \"content\": \"用一句话确认本地 API 工作正常。\"}
    ],
    \"max_tokens\": 64,
    \"temperature\": 0.7,
    \"enable_thinking\": false
  }"
```

测试结果如下图：

![alt text](/image/m1_test.png)

## 05｜实测性能：瓶颈很明确，就是 8 token/s 左右

所有数据都来自 4-bit、MLX-VLM、关闭 KV Cache 量化后的最终测试。为了不把偶然的冷启动当成常态，我把模型加载、首 token 和稳定生成拆开看。

| 指标 | 实测结果 |
|---|---:|
| 模型和处理器加载 | 约 4.5～5.3 秒 |
| 128 token 热启动首 token | 约 3.2 秒 |
| 稳定生成速度 | 约 8～8.7 token/s |
| 短上下文峰值内存 | 约 18.5～18.9GB |
| 2048 token 输入峰值内存 | 约 21.7GB |

短对话时，API 实测生成 128 个 token，速度是 **8.68 token/s**，峰值内存约 **18.55GB**。连续聊起来，每秒八九个 token 的体感不算丝滑，但回答是持续往外出的，不会慢到没法用。

长上下文更考验的是首 token。2048 token 输入时，prefill 大约 41.7 token/s，但等到第一个输出已经用了约 50 秒；4096 token 输入时接近 94 秒。也就是说，这台机器适合日常短到中等上下文，不适合每轮都塞几万 token 的大型代码仓库。

## 06｜MTP 也关掉：这份 4-bit 权重没有可用的 MTP 头

最后一个坑是 MTP。

模型配置里虽然能看到 `mtp_num_hidden_layers=1`，但我检查了下载下来的 4-bit 权重索引，没有找到可用的 `mtp`、额外预测层或 NextN 权重。API 返回的草稿解码信息也是空的：没有 draft model、没有 draft rounds，也没有 accepted tokens。

所以这套部署里，我选择关闭 MTP：不配置 `--draft-model`，不配置 `--draft-kind`，就走标准自回归解码。

最后还要提醒一句：这是 Uncensored 版本，模型本身没有可靠的安全护栏。
---

参考：

- [Qwen3.8-27B-Uncensored-MLX 模型页](https://huggingface.co/orcarouter/Qwen3.8-27B-Uncensored-MLX)
- [MLX-VLM 项目](https://github.com/Blaizzy/mlx-vlm)
