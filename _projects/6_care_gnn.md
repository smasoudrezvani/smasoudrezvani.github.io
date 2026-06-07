---
layout: page
title: CARE-GNN Reconstruction
description: Reproducing NeurIPS 2021 graph-based fraud detection from scratch
img: assets/img/caregnn_preview.jpg
importance: 6
category: work
---

Fraudsters actively camouflage their behaviour to evade graph-based detection systems. CARE-GNN (NeurIPS 2021) addresses this with a similarity-aware graph neural network that adapts its aggregation mechanism to distinguish camouflaged fraudsters from genuine users.

I reconstructed the full CARE-GNN architecture from scratch in PyTorch, without referencing any existing third-party implementation. The project covered the complete pipeline: dataset ingestion, heterogeneous graph construction, label-aware similarity estimation, and the reinforcement-learning-based neighbour selector that drives the model's camouflage-awareness.

Reproducing the paper surfaced several subtle implementation choices — particularly around the camouflage-aware aggregation strategy and the RL exploration-exploitation tradeoff for neighbour selection — that are underspecified in the original text. Findings and failure modes are documented in detail to support future researchers working on graph-based fraud detection.
