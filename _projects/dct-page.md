---
layout: page
title: DCT-Page
description: Training-free sparse attention that keeps long-context LLM decoding fast — while staying close to full-attention accuracy.
img: assets/img/dct-page.png
importance: 1
category: work
related_publications: false
---

**[View on GitHub →](https://github.com/Yoongon-Kim/DCT-Page)** · _ongoing_

Long-context decoding is bottlenecked by memory, not compute: every token the model generates
re-reads the entire KV cache, which can run to tens of gigabytes at 128K context. DCT-Page makes
that read sparse — attending only to the pages that matter — and holds decode cost roughly constant
past a context threshold, with no retraining, no architecture change, and no added weights. It began
as my undergraduate thesis at Seoul National University and is ongoing.

## How it works

A contiguous page of attention keys is overwhelmingly low-frequency: a discrete cosine transform
over a 32-token page concentrates ~85% of the keys' energy in the first four bins. The DCT is
orthonormal, so by Parseval, keeping the leading low-frequency coefficients preserves almost all of
each key's energy — only the small high-frequency residual is dropped. And since a query's
page-selection score is just its inner product with the page's keys, dropping that small residual
changes the score by at most a correspondingly small amount: the score barely moves. Each page is
therefore summarized by a compact **DCT-lowpass-IDCT proxy**: one small fixed matrix multiply, no
training, works on any frozen model.

At each decode step the query scores pages against their proxies, keeps the **top-k** (plus an
attention sink and a recent window) at full precision, and drops the rest; prefill stays fully
dense, so accuracy is only ever traded at decode time. A fixed 2,048-token budget per step keeps
decode cost near-constant. The per-step custom work is just **two Triton kernels** — one scores the
pages, one selects and packs the top-k — which hand the chosen page indices to **FlashInfer's paged
decode-attention kernel**; FlashInfer gathers the selected KV and computes attention in one shot, so
there is no separate assembly pass. A third Triton kernel builds a page's DCT proxy when the page
fills — only about once every 32 steps, far less often than the other two. The decode loop is CUDA-graph-capturable, so
DCT-Page rides production paged-attention throughput instead of competing with it.

## Results

Measured on two open-weight 8B models (Llama-3.1-8B-Instruct, Qwen3-8B) on a single NVIDIA A6000,
against four sparse-attention baselines. At a matched 2,048-token budget, DCT-Page beats the
strongest page-based baseline (Quest) by **+5.16** points on Llama and **+6.40** on Qwen3, and stays
within **0.07** points of full dense attention on LongBench v1.

| Model        | Method       | RULER | LongBench v1 | LongBench v2 |
| :----------- | :----------- | ----: | -----------: | -----------: |
| Llama-3.1-8B | Full-KV      | 88.78 |        50.32 |        29.82 |
|              | Quest        | 81.10 |        48.35 |        18.29 |
|              | Multipole    | 89.19 |        50.57 |        28.23 |
|              | **DCT-Page** | 86.26 |        50.27 |        30.02 |
| Qwen3-8B     | Full-KV      | 88.83 |        48.70 |        29.64 |
|              | Quest        | 77.32 |        47.77 |        18.09 |
|              | Multipole    | 88.04 |        48.48 |        31.41 |
|              | **DCT-Page** | 83.72 |        48.63 |        32.01 |

_All sparse methods run at a matched 2,048-token decode budget. RULER at 32K (average over 13
tasks); LongBench v1 (average over 6 tasks); LongBench v2 (overall accuracy). Full-KV is the dense
reference; Multipole is the cluster-based method — comparably accurate, but ~3.6× slower at 32K._

- **Robustness at length.** The LongBench v2 column above already matches or edges out dense
  attention — and it holds at extreme length: on the longest context bucket Quest's scorer collapses
  to near-random (0.00 / 0.93%), while DCT-Page is the only sparse method evaluated that stays
  functional.
- **Why it works.** At a matched 2,048-token budget, DCT-Page routes **~84%** of the dense softmax
  mass to the pages it keeps, against Quest's **65.6%** — an ~18-point gap in the quantity that
  directly bounds attention error.
- **Speed.** At 128K context, DCT-Page is **5.65× faster** than dense attention in the attention
  kernel and **1.70× faster** end-to-end. Against the cluster-based Multipole Attention — which is
  comparably accurate — it decodes ~3.6× faster at 32K.

It isn't uniformly best. On word-frequency and aggregation tasks, where the relevant evidence is
spread across many pages instead of concentrated on a few, any fixed-budget page selector — DCT-Page
included — gives up ground to dense attention. The thesis frames this as a structural limit of the
page-selection paradigm rather than of the proxy itself.

## What's in the repo

- The DCT-Page implementation — page compression, proxy scoring, and top-k selection as custom
  **Triton kernels**, wired into **FlashInfer**'s paged decode-attention.
- Four baselines for head-to-head comparison: SeerAttention-R, Multipole, Quest, and InfLLM.
- Diagnostics that measure, against a full-attention oracle, how well the cheap proxy recalls the
  pages that actually carry the attention mass.
- Benchmark runners for **RULER, LongBench v1/v2, AIME25, and GPQA** on Llama 3.1 and Qwen3.
