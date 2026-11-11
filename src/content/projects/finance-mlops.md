---
title: Finance MLOps <2026>
summary: A local, end-to-end MLOps pipeline for financial-news sentiment, from ingestion and training to serving, monitoring and retraining.
date: 2026-09-23
images: []
repository: https://github.com/y2kr/finance-mlops
---

## Overview

A local MLOps project that classifies financial headlines into three sentiment classes. The focus is the lifecycle around the model: reproducible data, automated pipelines, model promotion and monitoring, rather than model complexity.

Dagster orchestrates news ingestion, weak labeling and TF-IDF training. MLflow tracks and versions models, while BentoML serves predictions. Monitoring with Evidently and NannyML can trigger retraining; challengers must pass evaluation checks before replacing the active model.

The stack runs through Docker Compose, with Postgres for data storage and Prometheus and Grafana for metrics.

## Pipeline architecture

<button class="demo-image" style="max-width: none; margin-block: 2rem" type="button" data-gallery-image data-full="/files/imgs/finance-mlops/architecture.svg" aria-label="Open Finance MLOps pipeline diagram">
  <img src="/files/imgs/finance-mlops/architecture.svg" alt="Dagster orchestrates RSS ingestion into Postgres, Ollama weak labelling, TF-IDF training and monitoring. DVC PhraseBank also feeds training. Models pass a frozen gold gate into MLflow Registry and BentoML; predictions return to Postgres. Evidently and NannyML feed Prometheus and Grafana, and trigger retraining after two breaches." loading="lazy" />
</button>

From the [repository README](https://github.com/y2kr/finance-mlops#readme). Select the diagram to enlarge it, or [open the full-size SVG](/files/imgs/finance-mlops/architecture.svg) to zoom in. [Mermaid source](/files/imgs/finance-mlops/architecture.mmd).

<!-- ## What I learned

yeah -->
