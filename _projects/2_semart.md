---
layout: page
title: SemArt Digital Heritage
description: Investigating texture bias in CNNs using historical art datasets
img: assets/img/semart_preview.jpg # Replace with a real image path
importance: 2
category: work
---

This project focused on the intersection of deep learning and digital heritage, utilizing PyTorch to process and analyze a massive dataset of historical art images. The primary research question investigated "texture bias" in standard Convolutional Neural Networks (CNNs). 

Deep learning models often rely heavily on surface-level visual patterns (textures) rather than grasping the underlying semantic meaning of an image. This presents a unique challenge in digital heritage, where historical artifacts exhibit varying degrees of wear and surface degradation.


To address this, I trained and benchmarked models including ResNet-50 and CLIP utilizing contrastive learning techniques (InfoNCE). By designing rigorous evaluation protocols, the project assessed how well these models generalize across diverse visual categories without overfitting to low-level textural artifacts.