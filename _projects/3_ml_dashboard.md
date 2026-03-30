---
layout: page
title: ML Interpretation Dashboard
description: Interactive visualization of feature importance using SHAP
img: assets/img/dashboard_preview.jpg # Replace with a real image path
importance: 3
category: work
---

As machine learning models become more complex, interpretability becomes critical for trust and deployment. I built and deployed an interactive dashboard designed specifically to demystify model predictions using SHAP (SHapley Additive exPlanations) values.

The project features a full-stack implementation:
* **Backend:** Developed end-to-end APIs utilizing FastAPI to handle model training, evaluation, and data serving.
* **Frontend:** An interactive interface allowing users to explore feature importance visuals dynamically.
* **Deployment:** The entire system is fully containerized using Docker, ensuring consistent environments and scalable deployment.

<!-- <div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/dashboard_ui.jpg" title="Dashboard Interface" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The primary dashboard interface, rendering real-time SHAP dependency plots and summary visualizations.
</div> -->