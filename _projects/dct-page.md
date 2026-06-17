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
each key's energy — only the small high-frequency residual is dropped. Each page is therefore
summarized by a compact **DCT-lowpass-IDCT proxy**: one small fixed matrix multiply, no training,
works on any frozen model.

At each decode step the query scores pages against their proxies, keeps the **top-k** (plus an
attention sink and a recent window) at full precision, and drops the rest; prefill stays fully
dense, so accuracy is only ever traded at decode time. A fixed 2,048-token budget per step keeps
decode cost near-constant, no matter how long the context grows.

## Results

Measured on two open-weight 8B models (Llama-3.1-8B-Instruct, Qwen3-8B) on a single NVIDIA A6000,
against four sparse-attention baselines.

| Model        | Method          |     RULER | LongBench v1 | LongBench v2 |
| :----------- | :-------------- | --------: | -----------: | -----------: |
| Llama-3.1-8B | Full-KV         |     88.78 |        50.32 |        29.82 |
|              | Quest           |     81.10 |        48.35 |        18.29 |
|              | InfLLM          |     58.98 |        48.72 |        26.04 |
|              | **DCT-Page**    | **86.26** |    **50.27** |    **30.02** |
| Qwen3-8B     | Full-KV         |     88.83 |        48.70 |        29.64 |
|              | Quest           |     77.32 |        47.77 |        18.09 |
|              | SeerAttention-R |     66.74 |        48.24 |    **32.60** |
|              | **DCT-Page**    | **83.72** |    **48.63** |        32.01 |

<div style="text-align: center;"><em>All sparse methods run at a matched 2,048-token decode budget. RULER at 32K (average over 13 tasks); LongBench v1 (average over 6 tasks); LongBench v2 (overall accuracy). Full-KV is the dense reference; the best sparse-method score in each column is in bold.</em></div>

<div style="text-align: center;">
  <img
    src="{{ '/assets/img/dct-page.png' | relative_url }}"
    alt="DCT-Page decode speedup over full-KV attention as context length grows"
    style="max-width: 100%; height: auto;" />
  <p style="margin-top: 0.5rem;"><em>Decode speedup over full-KV (dense) attention as context length grows — up to 5.65× in the attention kernel and 1.70× end-to-end at 128K, while full-KV runs out of memory at the longest contexts.</em></p>
</div>

It isn't uniformly best. On word-frequency and aggregation tasks, where the relevant evidence is
spread across many pages instead of concentrated on a few, any fixed-budget page selector — DCT-Page
included — gives up ground to dense attention. That's a structural limit of the page-selection
paradigm itself, not of the proxy.

## What's in the repo

- The DCT-Page implementation — page compression, proxy scoring, and top-k selection as custom
  **Triton kernels**, wired into **FlashInfer**'s paged decode-attention.
- Four baselines for head-to-head comparison: SeerAttention-R, Multipole, Quest, and InfLLM.
- Diagnostics that measure, against a full-attention oracle, how well the cheap proxy recalls the
  pages that actually carry the attention mass.
- Benchmark runners for **RULER, LongBench v1/v2, AIME25, and GPQA** on Llama 3.1 and Qwen3.
