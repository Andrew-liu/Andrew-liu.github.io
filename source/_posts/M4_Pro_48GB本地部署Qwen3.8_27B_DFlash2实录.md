---
title: 我在 48GB M4 Pro 上给 Qwen3.8-27B 装上 DFlash2：实测最高 31.3 token/s
date: 2026-08-21 12:00:00
tags: [Qwen3.8, DFlash2, MLX, Apple Silicon, 本地部署]
---

这两天刚在 M1 Pro 上跑完 Qwen3.8-27B，普通 4-bit 推理大约 8～8.7 token/s。一直想着换个更牛逼的设备，今天在家里翻了下电脑，找到了我还有一台 48GB 的 M4 Pro（我家有四台mac，配置我真搞不清楚了）
又跑起来了标准4bit版本已经能稳定在 14.8 token/s 左右，但看到L总发的 DFlash 2 相关优化，我还是想试一下：再加一个草稿模型，到底能把本地生成速度推到什么程度？

最后的结果比我预想得更明显。相同提示词和采样参数下，数学推理从 14.78 token/s 提升到 31.33 token/s，代码生成从 14.76 提升到 26.15 token/s；三轮 GSM8K 测试里，DFlash 2 也跑到了 28.27 token/s，整体加速 1.91 倍。

这不是换了一个更小、更笨的模型来冒充 Qwen3.8-27B。DFlash 2 只是负责并行起草 token，最后仍由 27B 主模型验证，因此不会因为草稿猜错就把错误内容直接放出去。代价很小：除了约 16GB 的主模型，只要多下载一个约 3.6GB 的草稿模型。

<!-- more -->

## 01｜DFlash2 不是另一个大模型

传统推测解码通常会准备一个更小的草稿模型，让它先生成一串候选 token，再交给大模型批量验证。问题是草稿模型本身往往也要一个 token 接一个 token 地生成，只是每一步更便宜。

DFlash 2 的思路更激进：草稿阶段不再自回归等待，而是一次并行预测一整个 token 块。每个位置会保留多组候选，再由一个轻量路径选择器找出更连贯的组合，最后交给 Qwen3.8-27B 验证。猜对的部分直接接受，猜错的位置退回主模型继续生成。

所以 `Qwen3.8-27B-DFlash2` 不能单独聊天。它是一个约 2B 参数的草稿模型，必须和 `Qwen/Qwen3.8-27B` 配套使用。官方模型卡也明确说明：贪心解码时输出与目标模型一致，采样时则保持目标模型原本的概率分布。

![alt text](image/dflash21.png)

## 02｜我的机器和最终组合

这次实测使用的是：

```text
机器：MacBook Pro
芯片：Apple M4 Pro
统一内存：48GB
系统架构：Apple Silicon
Python：3.12
DFlash：0.1.0
MLX：0.32.0
MLX-LM：0.31.3
```
![alt text](image/dflash23.png)


模型组合如下：

| 角色 | 模型 | 本地占用 |
|---|---|---:|
| 目标模型 | mlx-community/Qwen3.8-27B-4bit | 约 16GB |
| 草稿模型 | incoai/Qwen3.8-27B-DFlash2 | 约 3.6GB |
| Python 虚拟环境 | DFlash + MLX 等依赖 | 约 472MB |

整套目录最好预留 25GB 以上。实际测试时，系统记录到的峰值内存占用约 20.7GB；48GB 机器余量很充足，模型加载和测试过程中没有发生 swap。至于 32GB 能不能获得同样表现，我没有在同一套配置上实测，不想只看模型文件大小就替大家下结论。

## 03｜单独建目录并安装 DFlash

我还是给模型单独建一个目录，环境、权重和测试文件都放在一起。以后升级或者删除，不会影响其他模型。

```bash
mkdir -p ~/Develop/AI/Qwen3.8-27B-MLX-DFlash2
cd ~/Develop/AI/Qwen3.8-27B-MLX-DFlash2

uv python install 3.12
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python "dflash[local]"
```

这里安装的是带本地推理依赖的版本，Apple Silicon 会走 MLX 后端。安装完先检查关键版本：

```bash
./.venv/bin/python -c \
  "import mlx, mlx_lm, importlib.metadata as m; print('dflash:', m.version('dflash'), 'mlx:', mlx.__version__, 'mlx-lm:', mlx_lm.__version__)"
```

我本机最终输出的是 `dflash 0.1.0`、`mlx 0.32.0` 和 `mlx-lm 0.31.3`。DFlash 还在快速更新，后续版本的参数和性能可能会变化，复现时最好把实际版本一起记下来。

## 04｜下载主模型和草稿模型

先创建模型目录，并把 Hugging Face 缓存也放进项目内：

```bash
mkdir -p models
export HF_HOME="$PWD/models/.hf-cache"
```

然后下载 MLX 4-bit 主模型：

```bash
./.venv/bin/hf download \
  mlx-community/Qwen3.8-27B-4bit \
  --local-dir ./models/Qwen3.8-27B-4bit
```

再下载 DFlash 2 草稿模型：

```bash
./.venv/bin/hf download \
  incoai/Qwen3.8-27B-DFlash2 \
  --local-dir ./models/Qwen3.8-27B-DFlash2
```

草稿模型下载下来约 3.6GB。启动时加上 `--draft-bits 4`，DFlash 会对草稿侧使用 4-bit 量化，减少统一内存占用。


## 05｜一条命令跑起来

下面是我最终使用的启动命令：

```bash
./.venv/bin/dflash generate mlx \
  --model ./models/Qwen3.8-27B-4bit \
  --draft ./models/Qwen3.8-27B-DFlash2 \
  --draft-bits 4 \
  --block-size 5 \
  --max-new-tokens 2048 \
  --reasoning xhigh \
  --temperature 0.7 \
  --top-p 0.95 \
  --top-k 20 \
  "计算 196 的所有正整数因数个数，并解释过程。"
```

几个参数需要单独解释一下。

`--reasoning xhigh` 是 Qwen3.8 的高推理强度。`--draft-bits 4` 对草稿模型做 4-bit 量化。`--block-size 5` 表示每轮推测和验证的块大小；DFlash 项目对 MLX 量化模型的建议也是不要超过 5，因为当前量化矩阵乘内核在更大的验证宽度下反而可能变慢。

块越大并不等于一定越快。草稿猜得越长，理论上一次能接受的 token 越多，但验证成本也会上升，而且后半段更容易猜错。真正适合自己机器的值，还是要跑 A/B 测试。

![alt text](image/dflash22.png)

## 06｜短测试最高跑到 31.3 token/s

我先写了三组固定提示词，分别测试短回答、数学推理和代码生成。每一组都先走普通自回归解码，再用同一个 4-bit 主模型走 DFlash 2；采样参数保持为 `temperature 0.7`、`top_p 0.95`、`top_k 20`，正式计时前还各做了一轮预热。

结果如下：

| 测试 | 普通解码 | DFlash 2 | 加速 |
|---|---:|---:|---:|
| 32-token 短回答 | 15.16 tok/s | 26.50 tok/s | 1.75× |
| 128-token 数学推理 | 14.78 tok/s | 31.33 tok/s | 2.12× |
| 128-token 代码生成 | 14.76 tok/s | 26.15 tok/s | 1.77× |

主模型和草稿模型一起加载约 3.62 秒。三组测试的首 token 时间都在 0.77～1.06 秒之间，DFlash 2 没有为了提高后续吞吐而制造特别夸张的首 token 等待。

数学题是这轮测试里的最高值，达到 31.33 token/s，超过普通解码两倍。代码生成稍低一些，说明加速效果会受到文本类型和草稿接受率影响，不应该只挑一个峰值就声称所有任务都能稳定翻倍。

## 07｜GSM8K 三轮测试：1.91 倍

短提示词容易受内容影响，我又从 GSM8K 测试集取了三道题，每题最多生成 128 token。`block-size 5` 下的结果是：

| 指标 | 实测结果 |
|---|---:|
| 普通解码吞吐 | 14.77 tok/s |
| DFlash 2 吞吐 | 28.27 tok/s |
| 整体加速 | 1.91× |
| 平均接受长度 | 3.74 |
| 峰值内存占用 | 约 20.7GB |

平均接受长度 3.74，可以理解为每次验证平均向前推进约 3.74 个输出 token。它没有把长度为 5 的草稿块次次吃满，但已经足以把生成速度从每秒 14.77 个 token 推到 28.27 个。

我还顺手比较了三个块大小：

| block size | DFlash 2 吞吐 | 相对普通解码 | 平均接受长度 |
|---|---:|---:|---:|
| 3 | 30.28 tok/s | 2.05× | 2.70 |
| 4 | 31.14 tok/s | 2.11× | 3.40 |
| 5 | 28.27 tok/s | 1.91× | 3.74 |

这组结果很有意思：`block-size 5` 的接受长度最高，但吞吐并不是最快；`block-size 4` 在我的 M4 Pro 上反而达到 31.14 token/s。原因也不复杂，多验证一个位置有额外计算成本，接受长度增加的收益没能完全覆盖它。

所以我的日常启动脚本仍保留官方更通用的 `block-size 5`，但如果只追求这台机器上的速度，当前测试里 `4` 更合适。换模型版本、提示词类型或 MLX 内核后，这个结论都可能变化。

## 08｜快了一倍，但不是没有代价

实际体感上，14.8 token/s 已经能顺畅聊天，28～31 token/s 则明显更接近云端服务持续吐字的速度。长回答、代码生成和 Agent 循环里，这个差异比短问短答更容易感知。

代价主要有三个。第一，多占约 3.6GB 磁盘，还要为草稿模型留运行内存。第二，部署链路比只加载一个 MLX 模型更复杂，DFlash 版本、草稿量化和块大小都会影响结果。第三，加速比不是常数，草稿模型越能猜中目标模型接下来的输出，收益越高；遇到接受率较低的内容，速度就会回落。

不过在 48GB M4 Pro 上，这笔交换我觉得很划算。它没有降低目标模型参数规模，也不是把回答偷偷换成小模型生成，而是在验证机制下减少 27B 主模型逐 token 解码的次数。
对已经能跑 Qwen3.8-27B、又觉得 15 token/s 还不够快的人，DFlash 2 确实值得装一次。

---

## 参考来源

1. [DFlash GitHub 仓库](https://github.com/z-lab/dflash)
2. [Qwen3.8-27B-DFlash2 模型卡](https://huggingface.co/incoai/Qwen3.8-27B-DFlash2)
3. [mlx-community/Qwen3.8-27B-4bit](https://huggingface.co/mlx-community/Qwen3.8-27B-4bit)
4. [DFlash 2 技术介绍](https://inco.ai/blog/dflash2/)
