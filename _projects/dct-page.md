---
layout: page
title: DCT-Page
description: Training-free sparse page attention that speeds up long-context LLM decoding.
img: assets/img/dct-page.png
importance: 1
category: work
related_publications: false
---

**[View on GitHub →](https://github.com/Yoongon-Kim/DCT-Page)** · _ongoing_

DCT-Page is a decode-time sparse-attention method for long-context LLMs. During autoregressive
decoding the KV cache grows linearly with the context, and attending over all of it becomes the
dominant cost. DCT-Page keeps that cost roughly constant — with no retraining.

## Idea

The KV cache is split into fixed-size pages (32 tokens by default). Each page is summarized by a
compact **DCT-lowpass-IDCT proxy** — a few low-frequency coefficients that approximate the page.
At every decoding step the query scores pages against their proxies, keeps the **top-k most
relevant pages** at full precision, and drops the rest. Prefill runs full attention unchanged, so
accuracy is only ever traded at decode time, where it is cheapest.

Because the proxy is a fixed linear transform, the scorer needs **no training** — it works on a
frozen model out of the box.

## What's in the repo

- The DCT-Page implementation with **fused Triton kernels** (scoring, top-k, KV assembly, RoPE) and
  PyTorch fallbacks.
- Five baselines for head-to-head comparison: SeerAttention-R, Multipole, Quest, DuoAttention, and
  InfLLM.
- An oracle / diagnostic suite for analyzing which pages actually matter.
- Benchmark runners for **RULER, LongBench v1/v2, AIME25, and GPQA** on Llama 3.1 and Qwen3.

This is active, ongoing work.
