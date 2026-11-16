# Research data

Original experiment files supporting the unpublished manuscript **Deploying Cross-Layer SVD KV-Cache Compression for Prefill-Decode Disaggregated Inference**.

All 663 experiment files are copied byte for byte from the experiment archives. JSON and JSONL files are the original outputs or configurations of the experiment programs. Their fields, paths, numerical precision, and records have not been rewritten. The package contains no newly generated table summaries, data indexes, or scripts.

The ZIP extracts the three data directories and the English and Chinese README documents directly into the destination directory.

## Contents

| Directory | Contents |
|---|---|
| `simulation/` | Original files from 105 simulation runs: timelines, request records, configurations, recorded commands, and backend logs for runs with activation preemption |
| `measurements/` | Original SVD, reconstruction, SplitZip, and MixHQ timing files, plus GPU operator profiles |
| `accuracy/` | Original result files from 11 configurations on ten RULER tasks, plus the available per-task metadata and recorded commands |

CSV files can be opened with spreadsheet or data-analysis software. JSON and JSONL files retain the formats written by the experiment programs. Any absolute paths in the original metadata describe the machine used for the experiments; they are not paths required for reading this package.

## Simulation results

Each `simulation/<run>/` directory contains `timeline.csv`, `requests.csv`, `config.json`, and `command.json`. The 15 runs with activation preemption also contain the original `backend.log` with `send_service` records.

The 105 runs comprise 100 configurations used in the network, rank, layout, and preemption experiments, plus five method-comparison runs. Directory names retain the original run IDs. For example:

- `paper_prefill_v1_net_bw1.25_lat10_k7`: reference raw-KV Baseline.
- `paper_prefill_v1_net_bw1.25_lat10_k0`: reference no-skip.
- `paper_prefill_v1_net_bw1.25_lat10_k1`: reference tail-1.
- `paper_prefill_v1_rank546` and `paper_prefill_v1_rank1092`: additional K-rank settings; V rank is 1.5 times K rank.
- `paper_prefill_v1_layout*`: layer-placement experiments; `local` and `gather` identify the two windowing schemes.
- `paper_prefill_v1_preempt*`: activation-preemption experiments.

The reference configuration uses Llama-3.2-3B-Instruct, BF16, 8192 input tokens, TP=1, PP=7, four layers per window, chunks of 2048 tokens, 1.25 GiB/s bandwidth, and 10 ms link latency.

`timeline.csv` contains the complete original timeline, including later decode events. Its columns are `instance`, `batch`, `rank`, `node_id`, `name`, `start_ns`, and `end_ns`; timestamps are in nanoseconds. For the paper's prefill results, select `instance=0`. The paper's TTFT is the end of the final-chunk (`batch=3`) `COMP_NODE_sampler` event. Prefill completion is the latest end of all `instance=0` events, including receiver reconstruction and sampler notifications. The `TTFT` field in the original `requests.csv` is not a direct substitute for this definition.

The simulation data report modeled latency, not measurements from a deployed multi-machine serving system.

## GPU measurements

The measured GPU is an NVIDIA GeForce RTX 4070 Ti SUPER.

| File |
|---|
| `measurements/svd/timings_legacy.json` |
| `measurements/reconstruction_fragments/fragment_timings_legacy.json` |
| `measurements/splitzip/timings.json` |
| `measurements/mixhq/timings.json` |
| `measurements/operator_profile/` |

The four timing JSON files retain the individual CUDA timing samples, experiment settings, and aggregate fields originally recorded by the measurement programs. The simulation uses CUDA-event medians. Operator-profile CSV files use microseconds; the codec timing fields named `cuda_ms` use milliseconds. Consult the original field names and `meta.yaml` for the profile configuration.

## Accuracy results

Each configuration evaluates 96 samples per task on ten RULER tasks.

| Directory | Configuration |
|---|---|
| `accuracy/rank_sweep/baseline/` | Uncompressed dense inference |
| `accuracy/rank_sweep/cr6/` | No-skip, K/V ranks 364/546 |
| `accuracy/rank_sweep/cr4/` | No-skip, K/V ranks 546/819 |
| `accuracy/rank_sweep/cr2/` | No-skip, K/V ranks 1092/1638 |
| `accuracy/tail_skip/skip1/` through `skip6/` | Tail-1 through tail-6, K/V ranks 546/819 |
| `accuracy/mixhq/<task>/` | MixHQ results for each task |

The rank-sweep and tail-skip JSONL files are incremental result records. Use the final line: `correct` contains all 96 sample scores and `avg_score` is the task mean. Do not treat the successive lines as independent sets of samples. The files also retain the original predictions and ground-truth answers. Their companion `.metadata.json` files contain the recorded sample metadata; `command.json` records the experiment invocation.

Each `accuracy/mixhq/<task>/mixhq.jsonl` instead has one record per sample, including `index`, `prediction`, `ground_truth`, and `score`. Scores are on a 0–1 scale. Multiply task means by 100 to obtain the paper's 0–100 scores; the macro-average is the equally weighted mean of the ten task means. Differences between scores on that scale are percentage points.

## Correspondence with the paper

| Paper material | Data to use |
|---|---|
| Table 1: method comparison | The five method-comparison directories in `simulation/`; SplitZip and MixHQ timing files |
| Tables 2–3: rank and computational cost | Reference no-skip and `rank546`/`rank1092` simulation runs; SVD timings; rank-sweep accuracy files |
| Table 4 and Tables B.1–B.2: accuracy | `accuracy/rank_sweep/`, `accuracy/tail_skip/`, and `accuracy/mixhq/` |
| Tables 5–6: window placement | `simulation/paper_prefill_v1_layout*/` and reconstruction-fragment timings |
| Tables 7–8: bandwidth and latency | `simulation/paper_prefill_v1_net_*/` |
| Table 9 and Tables C.1–C.2: preemption | `simulation/paper_prefill_v1_preempt*/` and the corresponding nonpreemptive reference runs |
| Figure 3 and Figures A.1–A.3: timelines | Reference-network runs with k=0, 1, 2, and 7, as appropriate |

The package contains experiment outputs and the accompanying original configurations, not the full implementation, model weights, evaluation prompt corpus, or captured KV tensors. Original accuracy results include predictions and ground-truth answers, but new inference requires the external inputs and implementation. No new license is granted by assembling these files; third-party rights remain with their respective owners.


## References and acknowledgments

This study builds on the following methods, simulator, and benchmark. Please also cite the original works when using their methods or software.

1. **xKV — cross-layer SVD compression.** Chi-Chih Chang et al. “xKV: Cross-Layer KV-Cache Compression via Aligned Singular Vector Extraction.” ICML, 2026. [Paper](https://proceedings.mlr.press/v306/chang26d.html) · [Code](https://github.com/abdelfattah-lab/xKV).
2. **LLMServingSim 2.0 — simulation infrastructure.** Jaehong Cho et al. “LLMServingSim 2.0: A Unified Simulator for Heterogeneous and Disaggregated LLM Serving Infrastructure.” ISPASS, 2026. [Paper](https://doi.org/10.1109/ISPASS69572.2026.00012) · [Code](https://github.com/casys-kaist/LLMServingSim).
3. **RULER — long-context evaluation benchmark.** Cheng-Ping Hsieh et al. “RULER: What's the Real Context Size of Your Long-Context Language Models?” COLM, 2024. [Paper](https://openreview.net/forum?id=kIoBbc76Sy) · [Code](https://github.com/NVIDIA/RULER).
4. **KVServe — source of the MixHQ comparison method.** Zedong Liu et al. “KVServe: Service-Aware KV Cache Compression for Communication-Efficient Disaggregated LLM Serving.” SIGCOMM, 2026. [Paper](https://doi.org/10.1145/3789240.3829139) · [Author manuscript](https://arxiv.org/abs/2605.13734v1) · [Code](https://github.com/hpdss-group/KVServe).
5. **SplitZip — lossless compression comparison method.** Yipin Guo and Siddharth Joshi. “SplitZip: Ultra Fast Lossless KV Compression for Disaggregated LLM Serving.” arXiv:2605.01708v3, 2026. [Paper](https://arxiv.org/abs/2605.01708v3) · [Code](https://github.com/Intelligent-Microsystems-Lab/SplitZip).
