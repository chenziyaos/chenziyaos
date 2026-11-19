### 陈子瑶 · Beijing

我在这里放两类东西：**大模型推理引擎的拆解笔记配套代码**（llama.cpp / vLLM / SGLang），和一个**让 agent 上下文可见、可编辑的画布工具**（Weft）。

*Companion code for my from-scratch walkthroughs of llama.cpp, vLLM and SGLang, plus Weft, a canvas for editing what an LLM agent reads next.*

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
