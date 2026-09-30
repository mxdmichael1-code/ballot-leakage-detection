# Experiment Notebooks

This directory contains the experiments used to develop and evaluate the ballot leakage detection approach.

The notebooks follow the progression from an off-the-shelf BGE embedding baseline to triplet-loss fine-tuning for PRO / CON / NEUTRAL semantic separation.

## Experiment Overview

| Notebook | Dataset | Purpose |
|---|---|---|
| `01_bge_baseline.ipynb` | v1 | Evaluate the original BGE embedding space and establish retrieval/classification baselines |
| `02_directional_finetuning.ipynb` | v1 | Test whether triplet-loss fine-tuning improves PRO vs. CON semantic separation |
| `03_threeway_finetuning.ipynb` | v1 | Extend fine-tuning to PRO / CON / NEUTRAL using a sampled set of neutral examples |
| `04_full_train_finetuning.ipynb` | v1 | Train the three-way approach on the full v1 training split |
| `05_expanded_dataset_finetuning.ipynb` | v2 | Evaluate the same three-way approach after expanding and rebuilding the dataset |

## Experimental Progression

### 01 — BGE Baseline

Tests `BAAI/bge-base-en-v1.5` without fine-tuning.

The baseline revealed that semantically similar but directionally opposite statements can occupy nearby regions of the original embedding space, motivating task-specific fine-tuning.

### 02 — Directional Fine-Tuning

Fine-tunes BGE using triplet loss on sentences annotated as either PRO or CON.

The objective is to pull a ballot sentence toward reference statements with the correct outcome direction and push it away from statements representing the opposite direction.

### 03 — Three-Way Fine-Tuning

Introduces NEUTRAL as a third semantic class.

Training uses PRO, CON, and a sampled subset of 60 NEUTRAL sentences to test whether the model can distinguish outcome leakage from ordinary feedback.

### 04 — Full-Train Fine-Tuning

Extends the three-way experiment to the complete v1 training split, including all available NEUTRAL examples.

### 05 — Expanded Dataset Fine-Tuning

Repeats the three-way methodology using Dataset v2, which includes additional manually annotated directional examples and a rebuilt judge-separated split.

Because Dataset v2 uses a different validation split, its metrics should not be directly compared with Dataset v1 as a controlled one-variable experiment.

## Common Setup

Unless otherwise stated, experiments use:

- **Embedding model:** `BAAI/bge-base-en-v1.5`
- **Embedding dimension:** 768
- **Model input:** Individual ballot sentence
- **Fine-tuning objective:** Triplet loss
- **Evaluation classes:** PRO / CON / NEUTRAL
- **Inference:** Maximum similarity to class-specific reference embeddings
- **Test split:** Held out during development

Dataset splits used by each notebook are stored in:

- `../data/splits_v1/`
- `../data/splits_v2/`

See `../data/README.md` for the annotation schema and dataset construction details.
