# Ballot Leakage Detection

An embedding-based system for detecting sentence-level inconsistencies between debate judge feedback and recorded ballot outcomes.

## Overview

Debate judges typically submit both a final vote (`PRO` or `CON`) and written feedback explaining their decision. In some ballots, the written feedback may implicitly or explicitly suggest an outcome that is inconsistent with the recorded vote.

For example:

> **Recorded vote:** PRO  
> **Written feedback:** "Con wins the key clash because Pro fails to respond to their argument."

Human review can identify these inconsistencies, but manually checking every sentence across hundreds of ballots is time-consuming.

This project explores whether sentence embeddings can provide a lightweight first-stage screening mechanism for identifying potential **ballot information leakage**.

## Approach

The system is inspired by the retrieval stage of RAG:

```text
Ballot
  ↓
Sentence-level chunking
  ↓
BGE embeddings
  ↓
Similarity against reference statements
  ↓
PRO / CON / NEUTRAL
```

Each feedback sentence is embedded using `BAAI/bge-base-en-v1.5` and compared against reference statements representing three semantic directions:

- **PRO** — feedback suggesting a PRO-favorable outcome
- **CON** — feedback suggesting a CON-favorable outcome
- **NEUTRAL** — feedback that does not reveal a clear outcome

The goal is not to predict the ballot from scratch, but to identify sentences that may reveal or contradict ballot information and surface them for further review.

## Why Fine-Tune the Embedding Model?

General-purpose embedding models are optimized for semantic similarity, but this task depends on small changes that can completely reverse the meaning of a ballot statement.

For example, the original BGE model produces high cosine similarity for pairs such as:

| Sentence Pair | Cosine Similarity |
|---|---:|
| "Pro wins the debate." / "Con wins the debate." | 0.9404 |
| "Pro has better weighing." / "Con has better weighing." | 0.9861 |
| "Con failed to answer Pro." / "Pro failed to answer Con." | 0.8904 |
| "Pro dropped Con." / "Con dropped Pro." | 0.9086 |
| "Judge votes Pro." / "Judge votes Con." | 0.9551 |

These sentences are topically and linguistically similar, but they imply opposite ballot outcomes.

To make the embedding space more sensitive to this distinction, BGE is fine-tuned using **triplet loss**:

```text
Anchor:    Real ballot sentence
Positive:  Reference statement with the correct direction
Negative:  Reference statement with an incorrect direction
```

Training encourages the anchor to move closer to the correct semantic reference and farther from incorrect references.

## Dataset

The dataset contains sentence-level annotations extracted from debate judge ballots.

Each sentence is annotated along two dimensions:

### Leakage Direction

| Label | Meaning |
|---|---|
| `PRO` | Sentence provides information favoring PRO |
| `CON` | Sentence provides information favoring CON |
| `NEUTRAL` | Sentence does not reveal a clear outcome |

### Leakage Strength

| Level | Meaning |
|---|---|
| `0` | No meaningful outcome leakage |
| `1` | Implicit directional signal |
| `2` | Explicit or decisive outcome information |

The current embedding experiments primarily train on **leakage direction**. Leakage strength is retained for annotation and diagnostic analysis rather than used as a direct training target.

See [`data/README.md`](data/README.md) for the full annotation schema and dataset documentation.

## Experiments

Four experiments compare the original BGE embedding space with task-specific fine-tuning.

| Experiment | Method | Fine-Tuned | Accuracy | Macro F1 | Direction Accuracy |
|---|---|---:|---:|---:|---:|
| `01A` | Directional BGE baseline | No | 0.5410 | 0.4813 | 0.3030 |
| `01B` | Three-way BGE baseline | No | 0.4262 | 0.4272 | 0.5152 |
| `02A` | Directional triplet fine-tuning | Yes | 0.5738 | 0.5619 | 0.6970 |
| `02B` | Three-way triplet fine-tuning | Yes | **0.7049** | **0.6892** | **0.6970** |

All four experiments use the same 61-sentence validation set. The held-out test set is not used during model development.

### Directional Formulation — 01A → 02A

The first formulation compares PRO and CON similarity scores and uses a neutral threshold to produce three-way predictions.

Triplet fine-tuning substantially improves the model's ability to distinguish PRO from CON directional leakage.

### Three-Way Formulation — 01B → 02B

The second formulation represents PRO, CON, and NEUTRAL as explicit semantic target classes.

This provides a cleaner comparison between untouched BGE and task-specific fine-tuning because both experiments use the same three-way target-bank inference structure.

See [`notebooks/README.md`](notebooks/README.md) for the individual experiments and [`results/README.md`](results/README.md) for detailed results.

## Current Findings

The experiments suggest two main findings:

**1. Generic semantic similarity is insufficient for ballot direction.**  
Untouched BGE places many role-reversed PRO/CON statements very close together despite their opposite implications.

**2. Task-specific triplet fine-tuning improves directional representation.**  
Fine-tuning substantially improves both three-way classification and PRO/CON direction discrimination under the evaluated validation set.

Remaining errors suggest that distinguishing **which side a directional sentence favors** remains more difficult than recognizing whether the sentence contains directional information at all.

## Repository Structure

```text
ballot-leakage-detection/
│
├── README.md
│
├── data/
│   ├── README.md
│   └── ballot_leakage_candidate_dataset - Candidate Dataset.csv
│
└── notebooks/
    ├── README.md
    ├── 01A_bge_baseline.ipynb
    ├── 01B_bge_threeway_baseline.ipynb
    ├── 02A_BGE_Directional_Finetuning.ipynb
    └── 02B_BGE_ThreeWay_Finetuning.ipynb
 
```

## Model and Training Setup

- **Base model:** `BAAI/bge-base-en-v1.5`
- **Embedding dimension:** 768
- **Input:** Individual ballot sentence
- **Similarity:** Cosine similarity
- **Fine-tuning objective:** Triplet loss
- **Classes:** PRO / CON / NEUTRAL
- **Epochs:** 2
- **Batch size:** 16
- **Learning rate:** 2e-5
- **Warmup ratio:** 0.1
- **Random seed:** 42

The model currently operates on sentences independently. Context such as the previous sentence, next sentence, ballot section, and recorded ballot outcome is not provided as model input.

## Limitations and Next Steps

The current dataset and validation set are relatively small, so results should be interpreted as experimental rather than as evidence of production-level performance.

Future work includes:

- evaluating the final methodology on the held-out test set;
- expanding the number and diversity of annotated ballots;
- comparing sentence-only inference with surrounding sentence context;
- investigating remaining PRO/CON polarity errors;
- evaluating escalation of ambiguous cases to an LLM or human reviewer.

The intended system is a **screening tool rather than an autonomous ballot reviewer**: embedding-based detection can identify potentially inconsistent sentences for more detailed review.
