---
layout: page
title: Multimodal DocVQA
description: Vision-Language Model for Document Question Answering
img: assets/img/docvqa_preview.jpg # Replace with a real image path
importance: 1
category: work
---

Document Question Answering (DocVQA) requires a system to understand both the textual content and the visual layout of a document simultaneously. In this AI seminar project, I built a multimodal pipeline designed to reason over complex document structures using both text and vision.

The architecture fuses visual embeddings extracted via ResNet and CLIP with text data obtained from OCR outputs. These combined inputs are fed into transformer-based Large Language Models (LLMs) to achieve multimodal alignment and generate accurate answers based on the document's layout and content.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/docvqa_architecture.jpg" title="Model Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    An overview of the multimodal pipeline, showcasing the integration of OCR text extraction with CLIP/ResNet visual embeddings.
</div>

The training workflows were implemented in PyTorch, focusing on optimizing the alignment between the visual and textual representations to improve the model's spatial reasoning capabilities.