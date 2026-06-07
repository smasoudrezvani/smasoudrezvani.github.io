---
layout: page
title: Algorithmic Trading System
description: ML research framework for XAUUSD and forex trading strategies
importance: 7
category: work
---

Systematic trading requires bridging raw price data, feature engineering, and reproducible experiment tracking — all with the rigour of a production ML system.

I architected an end-to-end research framework targeting XAUUSD and major forex pairs, running from September 2024 to September 2025. The pipeline uses Polars for high-performance feature engineering over price-action patterns (support/resistance levels, volume profiles, candlestick structures) and feeds into scikit-learn and XGBoost models for signal generation.

All experiments are tracked with Weights & Biases, enabling systematic ablations across feature sets, model architectures, and risk parameters. The system is designed for reproducibility: seeded train/test splits, pinned dependencies, and data versioning ensure that any historical backtest can be exactly reproduced — a discipline imported directly from ML research practices into a trading context.
