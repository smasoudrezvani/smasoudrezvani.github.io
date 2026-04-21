---
layout: page
title: S3-MinIO Starter Kit
description: Local S3-compatible storage and AWS EC2 deployment
img: assets/img/minio_preview.jpg # Replace with a real image path
importance: 5
category: work
---

Testing data pipelines against cloud storage can quickly accrue unnecessary costs during the development phase. To solve this, I built a local, S3-compatible storage system utilizing Docker and MinIO.

This starter kit allows developers to architect, test, and debug heavy data pipelines locally with the exact same API calls they would use in production. Once validated locally, I extended the infrastructure to deploy seamlessly onto AWS EC2 instances, providing a sandbox to explore real-environment behavior and latency without the overhead of standard S3 testing.
