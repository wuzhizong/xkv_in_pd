# 研究数据

本数据包用于支持待发表论文 **Deploying Cross-Layer SVD KV-Cache Compression for Prefill-Decode Disaggregated Inference**。

663 个实验文件均从实验归档逐字节复制。JSON 和 JSONL 文件是实验程序原本生成的结果或配置，未改写其中的字段、路径、数值精度或记录。数据包不包含另外生成的表格汇总、数据索引或脚本。

ZIP 解压后，会在目标目录下直接生成三个数据目录，以及中英文 README。

## 目录内容

| 目录 | 内容 |
|---|---|
| `simulation/` | 105 次仿真的原始时间线、请求记录、配置、命令记录，以及启用激活抢占时的后端日志 |
| `measurements/` | SVD、重建、SplitZip 和 MixHQ 的原始计时文件，以及 GPU 算子性能测量文件 |
| `accuracy/` | 11 种配置在十个 RULER 任务上的原始结果文件，以及已有的逐任务元数据和命令记录 |

CSV 文件可用电子表格软件或数据分析工具打开。JSON 和 JSONL 保留实验程序写入时的格式。原始元数据中的绝对路径描述的是实验机器上的位置，读取本数据包不依赖这些路径。

## 仿真结果

每个 `simulation/<run>/` 目录包含 `timeline.csv`、`requests.csv`、`config.json` 和 `command.json`。15 次启用激活抢占的运行还保留了原始 `backend.log`，其中含有 `send_service` 记录。

105 次运行包括网络、秩、布局和抢占实验中的 100 个配置，以及五种方法的对比运行。目录名保留原始运行 ID，例如：

- `paper_prefill_v1_net_bw1.25_lat10_k7`：参考网络下传输原始 KV 的 Baseline。
- `paper_prefill_v1_net_bw1.25_lat10_k0`：参考网络下的 no-skip。
- `paper_prefill_v1_net_bw1.25_lat10_k1`：参考网络下的 tail-1。
- `paper_prefill_v1_rank546` 和 `paper_prefill_v1_rank1092`：额外的 K 秩设置，V 秩为 K 秩的 1.5 倍。
- `paper_prefill_v1_layout*`：层布局实验；`local` 和 `gather` 对应两种窗口划分方案。
- `paper_prefill_v1_preempt*`：激活抢占实验。

参考配置为 Llama-3.2-3B-Instruct、BF16、8192 个输入 token、TP=1、PP=7、每个窗口四层、每个 chunk 2048 个 token、带宽 1.25 GiB/s、链路延迟 10 ms。

`timeline.csv` 保留完整原始时间线，包括后续 decode 事件。字段为 `instance`、`batch`、`rank`、`node_id`、`name`、`start_ns` 和 `end_ns`，时间戳单位为纳秒。读取论文的 prefill 结果时，选择 `instance=0`。论文中的 TTFT 是最后一个 chunk（`batch=3`）的 `COMP_NODE_sampler` 事件结束时间；prefill 完成时间是全部 `instance=0` 事件中的最晚结束时间，包括接收端重建和 sampler 通知。原始 `requests.csv` 中名为 `TTFT` 的字段不能直接替代这一定义。

仿真数据报告的是模型化延迟，不是实际部署的多机推理服务系统的实测延迟。

## GPU 测量

实测 GPU 为 NVIDIA GeForce RTX 4070 Ti SUPER。

| 文件 |
|---|
| `measurements/svd/timings_legacy.json` |
| `measurements/reconstruction_fragments/fragment_timings_legacy.json` |
| `measurements/splitzip/timings.json` |
| `measurements/mixhq/timings.json` |
| `measurements/operator_profile/` |

四个计时 JSON 保留了测量程序原本记录的逐次 CUDA 计时样本、实验设置和汇总字段。仿真采用 CUDA event 计时的中位数。算子测量 CSV 的时间单位为微秒，编解码计时中名为 `cuda_ms` 的字段以毫秒为单位。具体测量配置见原始字段和 `meta.yaml`。

## 准确率结果

每种配置评测十个 RULER 任务，每个任务 96 个样本。

| 目录 | 配置 |
|---|---|
| `accuracy/rank_sweep/baseline/` | 未压缩的 dense 推理 |
| `accuracy/rank_sweep/cr6/` | No-skip，K/V 秩 364/546 |
| `accuracy/rank_sweep/cr4/` | No-skip，K/V 秩 546/819 |
| `accuracy/rank_sweep/cr2/` | No-skip，K/V 秩 1092/1638 |
| `accuracy/tail_skip/skip1/` 至 `skip6/` | Tail-1 至 tail-6，K/V 秩 546/819 |
| `accuracy/mixhq/<task>/` | 各任务的 MixHQ 结果 |

秩扫描和尾部跳过实验的 JSONL 是逐步累积的结果记录。读取时使用最后一行：`correct` 包含全部 96 个样本的得分，`avg_score` 是该任务的平均得分。不要把连续各行当成互相独立的样本集合。文件还保留原始预测和标准答案。配套的 `.metadata.json` 包含已记录的样本元数据，`command.json` 记录实验调用命令。

`accuracy/mixhq/<task>/mixhq.jsonl` 则是每行对应一个样本，包含 `index`、`prediction`、`ground_truth` 和 `score` 等字段。得分范围为 0–1；任务平均分乘以 100 后得到论文使用的 0–100 分数。宏平均是十个任务平均得分的等权平均，该分数尺度上的差值以百分点表示。


## 与论文的对应关系

| 论文内容 | 对应数据 |
|---|---|
| 表 1：方法对比 | `simulation/` 中五种方法的对比运行目录；SplitZip 和 MixHQ 计时文件 |
| 表 2–3：秩与计算开销 | 参考 no-skip、`rank546`／`rank1092` 仿真运行；SVD 计时；秩扫描准确率文件 |
| 表 4、表 B.1–B.2：准确率 | `accuracy/rank_sweep/`、`accuracy/tail_skip/` 和 `accuracy/mixhq/` |
| 表 5–6：窗口布局 | `simulation/paper_prefill_v1_layout*/` 和重建片段计时 |
| 表 7–8：带宽与延迟 | `simulation/paper_prefill_v1_net_*/` |
| 表 9、表 C.1–C.2：抢占 | `simulation/paper_prefill_v1_preempt*/` 及对应的非抢占参考运行 |
| 图 3、图 A.1–A.3：时间线 | 按图示策略选取参考网络下 k=0、1、2、7 的运行 |

数据包包含实验输出和配套的原始配置，不包含完整实现、模型权重、评测 prompt 语料或采集的 KV 张量。原始准确率结果包含预测和标准答案，但重新执行推理仍需外部输入及相应实现。汇集这些文件本身不授予新的许可，第三方权利仍归各自权利人所有。



## 参考文献与致谢

本研究基于以下方法、仿真器和评测基准开展。使用相关方法或软件时，也请引用相应的原始工作。

1. **xKV：跨层 SVD 压缩方法。** Chi-Chih Chang 等. “xKV: Cross-Layer KV-Cache Compression via Aligned Singular Vector Extraction.” ICML, 2026. [论文](https://proceedings.mlr.press/v306/chang26d.html) · [代码](https://github.com/abdelfattah-lab/xKV)。
2. **LLMServingSim 2.0：仿真基础设施。** Jaehong Cho 等. “LLMServingSim 2.0: A Unified Simulator for Heterogeneous and Disaggregated LLM Serving Infrastructure.” ISPASS, 2026. [论文](https://doi.org/10.1109/ISPASS69572.2026.00012) · [代码](https://github.com/casys-kaist/LLMServingSim)。
3. **RULER：长上下文评测基准。** Cheng-Ping Hsieh 等. “RULER: What's the Real Context Size of Your Long-Context Language Models?” COLM, 2024. [论文](https://openreview.net/forum?id=kIoBbc76Sy) · [代码](https://github.com/NVIDIA/RULER)。
4. **KVServe：MixHQ 对比方法的来源。** Zedong Liu 等. “KVServe: Service-Aware KV Cache Compression for Communication-Efficient Disaggregated LLM Serving.” SIGCOMM, 2026. [论文](https://doi.org/10.1145/3789240.3829139) · [作者稿](https://arxiv.org/abs/2605.13734v1) · [代码](https://github.com/hpdss-group/KVServe)。
5. **SplitZip：无损压缩对比方法。** Yipin Guo 和 Siddharth Joshi. “SplitZip: Ultra Fast Lossless KV Compression for Disaggregated LLM Serving.” arXiv:2605.01708v3, 2026. [论文](https://arxiv.org/abs/2605.01708v3) · [代码](https://github.com/Intelligent-Microsystems-Lab/SplitZip)。
