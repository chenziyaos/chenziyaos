### 陈子瑶 · SRE · Beijing

我是一名 SRE，日常工作是线上稳定性。最近在系统地学大模型推理引擎，也在写让 agent 帮忙做稳定性分析的工具。

*SRE working on reliability day to day; currently digging into LLM inference engines and agent tooling.*

## 日常在做什么

- **容灾与多活**：设计和执行机房级容灾演练（断网、切流、故障注入），演练结束后写复盘，把问题定级、追根因、跟到整改关闭。
- **存储高可用治理**：盘点数据库、缓存、KV 存储的副本数和跨机房分布，找出单点和不满足多活要求的实例，推动补副本、调整部署。
- **稳定性数据**：把「标准要求是多少」和「现网实际是多少」放在一起对比，按区域、按机房拆开看差距，不把不同区域合成一个总数。
- **让 agent 接手重复的分析**：把取数、对账、写复盘初稿这类重复劳动做成 agent 能调用的工具，人只做判断。[Weft](https://github.com/chenziyaos/weft) 就是在这个过程中做的：长时间的 agent 分析跑偏时，用它看清并剪掉 agent 正在读的过时上下文。

更早几年做的是 DevOps：Docker、Kubernetes、Go、Nginx（下面「不再维护」里的仓库是那时候的）。

## 为什么看 LLM 推理

大模型推理服务也要有人保障稳定性。我拆引擎源码，是为了在上线前回答三个问题（章节号以 zero-to-vllm 为准）：

| 问题 | 在哪一章算 |
|---|---|
| **容量**：一个模型、一种卡、一个平均上下文长度，KV 缓存能撑多少并发？ | 推理账本（`02-ledger`）。例：Qwen3-8B、单张 H100、显存利用率 0.92、每个请求 2048 token，算下来最多 197 个并发 |
| **过载时怎么退化**：KV 缓存满了，调度器是排队还是抢占？首 token 延迟（TTFT）和逐 token 延迟（ITL）怎么变？ | 调度器、chunked prefill（`05`、`06`） |
| **怎么部署**：张量并行（TP）切几份、选哪种量化、选哪种卡？ | 并行、量化、部署实战（`11`、`12`、`23`、`24`） |

模拟器的数字都要和引擎启动日志里的实测值对得上才算数。

## 现在在维护的仓库

| 仓库 | 它做什么 | 在哪能跑 |
|---|---|---|
| [**weft**](https://github.com/chenziyaos/weft) | 无限画布上的对话：每次问答是一个节点，节点之间的连线就是下一轮的上下文，剪断一根线，下一轮就读不到那段历史。可以接 Claude Code / Codex 这类终端 agent。[ThoughtDAG](https://github.com/chenxiachan/thoughtdag) 的 fork（MIT） | Node.js ≥ 20，本地 Web 应用 + 一个小代理 |
| [**zero-to-vllm**](https://github.com/chenziyaos/zero-to-vllm) | 按章拆 vLLM v0.32.0（24 个章节目录）：显存账本、调度与抢占、Paged KV、prefix caching、PD 分离、部署实战，每章带模拟器或实验脚本 | `sim_*.py` 在 CPU 上跑（Python ≥ 3.11 标准库，个别用 numpy）；`gpu_*.py` 要 GPU |
| [**zero-to-sglang**](https://github.com/chenziyaos/zero-to-sglang) | 按章拆 SGLang v0.5.22（23 章）：内存池、RadixAttention、overlap 与 CUDA Graph、HiCache、PD 分离 | 同上，CPU 练习需要 Python ≥ 3.10 + numpy |
| [**zero-to-llama**](https://github.com/chenziyaos/zero-to-llama) | 按章拆 llama.cpp（21 章）：GGUF/GGML 结构、内存与带宽预测器、量化对照、CPU 调优、投机解码，数字来自一台 4 vCPU / 8 GB 无 GPU 的机器 | Python ≥ 3.9 标准库，不装 `gguf` 包也能读 GGUF |

## 30 秒试一下

不需要 GPU，算一张 H100 上跑 Qwen3-8B 时 vLLM 能分到多少 KV 缓存：

```bash
git clone https://github.com/chenziyaos/zero-to-vllm.git
cd zero-to-vllm/02-ledger
python3 sim_ledger.py --model qwen3-8b --gpu h100-sxm
```

期望输出的开头：

```text
Qwen3-8B on 1 x H100 SXM  weight=bf16 kv=bf16 util=0.92
  requested          72.86 GiB  (总显存 x util)
  - 权重             15.26 GiB
  ...
  = KV 可用          55.43 GiB  ← 对照日志 Available KV cache memory
```

想直接用 Weft：

```bash
git clone https://github.com/chenziyaos/weft.git && cd weft
npm install
npm run server   # 终端 1：模型代理，http://localhost:3001
npm run dev      # 终端 2：画布，http://localhost:5173
```

## 使用前要知道的

- **zero-to-\* 是学习用的配套代码，不是库**。模拟器复现的是引擎主干里的关键账本，结果要和真实日志对账才算数；每个仓库 README 写了版本基线（具体到 commit），换版本后数字可能对不上。
- **Weft 没有发布到 npm**，CLI 需要自己 `npm run cli:pack` 打包安装；画布数据存在浏览器 IndexedDB 里。限制和排障见它的 [README](https://github.com/chenziyaos/weft#limitations)。
- **2015–2020 的仓库不再维护**：[ngx_stream_upstream_check_module](https://github.com/chenziyaos/ngx_stream_upstream_check_module)（Nginx stream 模块的 TCP 健康检查）、[onssh](https://github.com/chenziyaos/onssh)（基于 SSH 的 Web 终端，Python 2.7 + Tornado）、[nginx-devops](https://github.com/chenziyaos/nginx-devops)，以及[旧博客](https://chenziyaos.github.io/)（运维 / Docker / Go 方向的存档）。

## 联系

对哪个仓库有问题，直接在那个仓库开 issue。
