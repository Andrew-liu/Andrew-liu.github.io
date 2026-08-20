---
title: 4060Ti 8GB本地部署Qwen3.8-27B实录
date: 2026-08-18 12:00:00
tags: [Qwen3.8, llama.cpp, 本地部署, RTX 4060 Ti 8GB]
---

![alt text](/image/4060qwen1.png)

昨天在 X 上刷到一条帖子：国外大佬 拿 RTX 4060 8GB 笔记本跑起了 Qwen3.8-27B，64K 上下文，生成速度大约 5 token/s。

我正好也有一张 4060 Ti 8GB，就照着他的参数装了一遍。中间踩了个不小的坑，模型能加载，速度却一度只有 0.19 token/s，半天蹦不出一句话。好在最后找到了原因。

现在这套配置稳定生成约 4.2 token/s，MTP 也正常工作。代价很明确：27B 模型不可能全塞进 8GB 显存，剩下的权重主要靠系统内存和 CPU。聊个天就巨慢了，更别说处理任务了，纯玩具。

<!-- more -->

## 我的机器

```text
系统：Windows 10 64位
CPU：AMD Ryzen 7 7800X3D
内存：32GB
显卡：NVIDIA RTX 4060 Ti 8GB
Python：3.12.10
PyTorch：2.13.0+cu130
ComfyUI：0.33.0
```

最终用了 llama.cpp b10218 的 CUDA 12.4 构建，模型是 Unsloth 的 `Qwen3.8-27B-IQ4_XS.gguf`，文件大小 14.63GiB。 https://huggingface.co/unsloth/Qwen3.8-27B-GGUF

启动后有 25 层进显存，K、V 缓存压成 Q4_0。显存会吃掉约 7.8GiB，只剩一百多 MiB。测试前我把其他吃 GPU 的程序都关了，不然很容易在加载时 OOM。

## 建目录

和上一篇跑MiniMax H3一样，在 `D:\AI\Qwen3.8-27B` 目录。如果你准备换个位置，只改第一行就行。

```powershell
$InstallRoot = 'D:\AI\Qwen3.8-27B'
$BinDir = Join-Path $InstallRoot 'bin'
$DownloadDir = Join-Path $InstallRoot 'downloads'
$ModelDir = Join-Path $InstallRoot 'models'

New-Item -ItemType Directory -Force `
  -Path $BinDir, $DownloadDir, $ModelDir | Out-Null
```

整个目录预留 25GB 比较稳。模型占了大头，压缩包和下载缓存还要再吃掉一些空间。

## 安装 llama.cpp

我用的是 b10218。llama.cpp 的版本更新很勤，后面的版本可能已经修掉本文遇到的问题，也可能改了参数。想原样复现，先别追新。

Windows + NVIDIA 显卡需要这两个包：

- `llama-b10218-bin-win-cuda-12.4-x64.zip`
- `cudart-llama-bin-win-cuda-12.4-x64.zip`

```powershell
$LlamaZip = Join-Path $DownloadDir 'llama-b10218-bin-win-cuda-12.4-x64.zip'
$CudartZip = Join-Path $DownloadDir 'cudart-llama-bin-win-cuda-12.4-x64.zip'

curl.exe -L --fail --retry 5 -C - `
  -o $LlamaZip `
  'https://github.com/ggml-org/llama.cpp/releases/download/b10218/llama-b10218-bin-win-cuda-12.4-x64.zip'

curl.exe -L --fail --retry 5 -C - `
  -o $CudartZip `
  'https://github.com/ggml-org/llama.cpp/releases/download/b10218/cudart-llama-bin-win-cuda-12.4-x64.zip'

Expand-Archive -LiteralPath $LlamaZip -DestinationPath $BinDir -Force
Expand-Archive -LiteralPath $CudartZip -DestinationPath $BinDir -Force

& (Join-Path $BinDir 'llama-server.exe') --version
```

最后检查一下版本。正常会看到版本号 10218 和 Windows x86_64 构建信息。

## 下载模型

模型仓库在这里：

<https://huggingface.co/unsloth/Qwen3.8-27B-GGUF>

文件名是 `Qwen3.8-27B-IQ4_XS.gguf`。Hugging Face 页面显示 15.7GB。

我一开始用 curl，跑了几 GB 后反复停流，连接断了好几次。后来换成 Hugging Face 自己的 Xet 下载器，一次完成。

![alt text](/image/4060qwen2.png)

## 启动参数

下面是我最后跑通的命令。末尾的 `--no-op-offload` 很重要，后面单独说。

```powershell
$Server = Join-Path $BinDir 'llama-server.exe'
$Model = Join-Path $ModelDir 'Qwen3.8-27B-IQ4_XS.gguf'

& $Server `
  -m $Model `
  -c 64000 `
  --host 127.0.0.1 `
  --port 8080 `
  -ngl 25 `
  -ctk q4_0 `
  -ctv q4_0 `
  --threads 6 `
  --threads-batch 8 `
  --spec-type draft-mtp `
  --spec-draft-n-max 2 `
  --spec-draft-p-min 0.7 `
  --no-op-offload
```

加载成功后，终端会出现：

```text
model loaded
listening on http://127.0.0.1:8080
```

这时打开 <http://127.0.0.1:8080> 就能聊天。OpenAI 兼容接口是：

```text
http://127.0.0.1:8080/v1
```

如果要给局域网设备使用，需要另外处理监听地址和鉴权，不建议直接裸奔。

![alt text](/image/4060qwen3.png)

## 那个差点让我放弃的坑

X 原帖的命令里没有 `--no-op-offload`。我第一次完全照抄，模型确实加载成功了，但提示处理只有约 0.19 token/s。这速度和废物一样。

日志里反复出现这两行：

```text
fused Gated Delta Net (chunked) is assigned to device CUDA0
fused Gated Delta Net (chunked) not supported, set to disabled
```

混合卸载时，部分属于 CPU 层的算子又被尝试搬到 GPU，跨设备执行把速度拖垮了。加上 `--no-op-offload`，让这些算子老老实实留在原设备，速度立刻恢复。

短基准结果：

| 测试 | 速度 |
|---|---:|
| llama-bench pp32 | 10.79 token/s |
| llama-bench tg8 | 4.23 token/s |

再跑聊天接口，预热后的成绩是：

| 项目 | 实测结果 |
|---|---:|
| 模型加载 | 约 17.2 秒 |
| 提示处理 | 约 4.79 token/s |
| 生成 | 约 4.21 token/s |
| MTP 接受率 | 88.9%，24/27 |
| 显存占用 | 约 7.8GiB |

我这张卡稳定在 4.2 token/s 左右，和原帖的 5 token/s 差得不多。

## 几个可能遇到的问题

### 加载时 OOM

先把浏览器硬件加速、游戏和其他 CUDA 程序关掉。还不行就把 `-ngl 25` 改成 `-ngl 24`，让 CPU 多接一层。速度会慢一点，总比启动不了强。

### 第一轮特别慢

第一次请求要建立计算图。我这边第一轮用了约 54 秒，第二轮降到约 16 秒。所以装好后别只问一句就下结论，至少跑两轮。

## 用下来是什么感觉

4 token/s 不算快。普通聊天能明显看到字一个个往外蹦。不用考虑跑长任务了。

但这次折腾仍然挺有意思：一张 8GB 显卡，加上 32GB 内存，确实把 27B 稠密模型跑起来了。MTP 的接受率接近九成，也比我原先预想的好。

这个攻略适合想本地泡一下模型体验一下的人，不适合真正拿来打个模型干活。

## 参考来源
- [X 原帖：RTX 4060 8GB + 16GB RAM 运行配置](https://x.com/analogalok/status/2089053511451541903)
- [Unsloth Qwen3.8-27B GGUF](https://huggingface.co/unsloth/Qwen3.8-27B-GGUF)
- [llama.cpp Releases](https://github.com/ggml-org/llama.cpp/releases)
