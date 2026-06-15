---
layout: page
title: SNU FastMRI Challenge 2024
description: Accelerated MRI reconstruction under an 8 GB-VRAM budget (team SuperFastMRI).
img: assets/img/snu-fastmri.png
importance: 2
category: work
related_publications: false
---

**[View on GitHub →](https://github.com/SuperFastMRI-snu/SNU-FastMRI-Challenge)**

The [2024 SNU FastMRI Challenge](https://fastmri.snu.ac.kr/), hosted by Prof. Jong-Ho Lee's LIST Lab
at Seoul National University, is a stricter take on [Facebook AI's FastMRI challenge](https://fastmri.org/):
reconstruct multi-coil brain MRI from undersampled k-space measurements, but under tight
constraints — **8 GB of VRAM, no pretrained models, a small training set, and a 3,000-second
inference limit** — across a wide range of acceleration factors. I competed as a two-person team,
**SuperFastMRI**, with Dongwook Kho.

## Our approach

We used a **mixture of experts**: four [Feature-Image (FI) VarNet](https://www.nature.com/articles/s41598-024-59705-0)
sub-models, each specialized for a specific range of acceleration factors. At inference, an input's
acceleration is computed and routed to the matching expert (or the nearest one).

FI-VarNet keeps the data-consistency update in feature space rather than k-space, so the high-level
features that the baseline E2E-VarNet loses when converting back to k-space are preserved across
cascades. To fit the 8 GB budget we dropped block-wise attention in exchange for more cascades and
deeper UNets.

Each expert is itself a small **ensemble** — the two checkpoints with the lowest validation loss,
averaged — which we found consistently improved leaderboard results.
