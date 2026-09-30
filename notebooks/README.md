# Experiment Notebooks

This directory contains the four main experiments used to develop and evaluate the ballot leakage detection approach.

The experiments compare the original BGE embedding space with task-specific triplet-loss fine-tuning under two formulations:

1. **Directional:** PRO vs. CON similarity, with a threshold used to identify NEUTRAL sentences.
2. **Three-way:** PRO, CON, and NEUTRAL represented as explicit semantic target classes.

## Experiment Overview

| Notebook | Method | Fine-Tuned | Accuracy | Macro F1 | Direction Accuracy |
|---|---|---:|---:|---:|---:|
| `01A_bge_baseline.ipynb` | Directional BGE baseline | No | 0.5410 | 0.4813 | 0.3030 |
| `01B_bge_threeway_baseline.ipynb` | Three-way BGE baseline | No | 0.4262 | 0.4272 | 0.5152 |
| `02A_BGE_Directional_Finetuning.ipynb` | Directional triplet fine-tuning | Yes | 0.5738 | 0.5619 | 0.6970 |
| `02B_BGE_ThreeWay_Finetuning.ipynb` | Three-way triplet fine-tuning | Yes | 0.7049 | 0.6892 | 0.6970 |

All four experiments are evaluated on the same 61-sentence validation set. The held-out TEST split is not used during model development.

## 01A — BGE Directional Baseline

Evaluates the original `BAAI/bge-base-en-v1.5` model without fine-tuning.

Each sentence is compared with PRO and CON reference statements. The difference between PRO and CON similarity scores determines direction, while a validation-selected threshold identifies sentences as NEUTRAL.

This experiment establishes the baseline for the directional formulation and illustrates the difficulty generic semantic embeddings have with role-reversed statements.

## 01B — BGE Three-Way Baseline

Evaluates untouched BGE using explicit PRO, CON, and NEUTRAL semantic target banks.

Each validation sentence is compared with all three target banks, and the class with the highest maximum cosine similarity is selected.

This provides the direct no-fine-tuning baseline for the three-way formulation used in Experiment 02B.

## 02A — Directional Fine-Tuning

Fine-tunes BGE using only training sentences already annotated as directional leakage (PRO or CON).

Each sentence is used as an anchor in triplets that pull its representation toward the correct directional target and push it away from the opposite target.

Inference retains the PRO-minus-CON similarity score and validation-selected neutral threshold used in the directional baseline.

## 02B — Three-Way Fine-Tuning

Extends triplet fine-tuning by representing NEUTRAL as an explicit third semantic class.

Training uses all available PRO and CON training sentences together with 60 sampled NEUTRAL sentences. Each sentence is trained against both incorrect semantic classes.

Inference directly selects the highest-scoring class across the PRO, CON, and NEUTRAL target banks without a neutral threshold.

## Experimental Comparisons

The experiments are designed around two direct comparisons:

**01A → 02A:** measures the effect of directional triplet fine-tuning while retaining the directional inference formulation.

**01B → 02B:** measures the effect of three-way triplet fine-tuning while retaining the explicit PRO / CON / NEUTRAL target-bank formulation.

The second comparison provides the cleaner evaluation of whether task-specific fine-tuning improves the embedding space for three-way ballot leakage detection.

## Common Setup

- **Base model:** `BAAI/bge-base-en-v1.5`
- **Embedding dimension:** 768
- **Model input:** Individual ballot sentence
- **Similarity:** Cosine similarity
- **Fine-tuning objective:** Triplet loss
- **Evaluation classes:** PRO / CON / NEUTRAL
- **Validation set:** 61 sentences
- **Directional validation examples:** 33
- **Test set:** Held out during development

See `../data/README.md` for the dataset and annotation schema and `../results/` for consolidated evaluation results.
