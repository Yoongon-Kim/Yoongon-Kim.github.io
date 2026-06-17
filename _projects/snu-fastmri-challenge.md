---
layout: page
title: SNU FastMRI Challenge 2024
description: Accelerated MRI reconstruction under an 8 GB-VRAM budget (team SuperFastMRI).
img: assets/img/fastmri-cover.jpg
importance: 2
category: work
related_publications: false
---

**[View on GitHub →](https://github.com/SuperFastMRI-snu/SNU-FastMRI-Challenge)**

The [2024 SNU FastMRI Challenge](https://fastmri.snu.ac.kr/), hosted by Prof. Jong-Ho Lee's
[LIST Lab](https://list.snu.ac.kr/home) at Seoul National University, is a stricter take on
[Facebook AI's FastMRI challenge](https://fastmri.org/): reconstruct multi-coil brain MRI from
undersampled k-space measurements, but under tight constraints — **8 GB of VRAM, no pretrained
models, a small training set, and a 3,000-second inference limit** — across a wide range of
acceleration factors. I competed as a two-person team, **SuperFastMRI**, with Dongwook Kho,
placing **4th on the public leaderboard**.

## Our approach

We took a **mixture-of-experts approach**: four
[Feature-Image (FI) VarNet](https://www.nature.com/articles/s41598-024-59705-0) experts, each
specialized for a band of acceleration factors, with a mask classifier that reads each input's
acceleration and routes it to the matching expert (or the nearest one).

<div style="text-align: center;">
  <img
    src="{{ '/assets/img/snu-fastmri.png' | relative_url }}"
    alt="A mask classifier routes each input to the FI-VarNet sub-model trained for its acceleration range"
    style="max-width: 67%; height: auto;" />
</div>

_Each input is routed by its acceleration to one of four FI-VarNet experts (acc 4-5, 6-7, 8-9,
10-11)._

FI-VarNet keeps the data-consistency update in feature space rather than k-space, so the high-level
features that the baseline E2E-VarNet loses when converting back to k-space are preserved across
cascades. To fit the 8 GB budget we dropped block-wise attention in exchange for more cascades and
deeper UNets.

Each expert uses **checkpoint ensembling** — we average the two best checkpoints from its training
run, which consistently helped on the leaderboard.

<div style="text-align: center;">
  <img
    src="{{ '/assets/img/fastmri-ensemble.png' | relative_url }}"
    alt="Each expert averages two checkpoints of the same FI-VarNet taken at different epochs"
    style="max-width: 67%; height: auto;" />
</div>

_Each expert averages two checkpoints from the same training run._

---

_Card image: an axial T2 brain MRI by Aaron G. Filler, MD, PhD —
[CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/), via
[Wikimedia Commons](https://commons.wikimedia.org/wiki/File:MRI_T2_Brain_axial_image.jpg)._
