# Experimental Results

This directory contains the evaluation results from the ballot leakage detection experiments.

## Dataset v1

| Experiment | Training Samples | Accuracy | Macro F1 | Direction Accuracy |
|---|---:|---:|---:|---:|
| Original BGE Baseline | 0 | 0.6136 | 0.4762 | 0.2778 |
| Directional Fine-Tuning | 83 | 0.6591 | 0.5515 | 0.3889 |
| Three-Way Fine-Tuning | 143 | 0.7045 | 0.6250 | 0.6111 |
| Full-Train Fine-Tuning | 212 | 0.7045 | 0.6351 | 0.5556 |

All Dataset v1 experiments are evaluated on the same 44-sentence validation split.

### Metrics

- **Accuracy** — proportion of all validation sentences classified correctly as PRO, CON, or NEUTRAL.
- **Macro F1** — average F1 across PRO, CON, and NEUTRAL, giving each class equal weight.
- **Direction Accuracy** — accuracy on directional sentences only (ground-truth PRO or CON).

## Dataset v2

| Experiment | Training Samples | Accuracy | Macro F1 | Direction Accuracy |
|---|---:|---:|---:|---:|
| Expanded Dataset Fine-Tuning | 285 | 0.6721 | 0.6530 | 0.6061 |

Dataset v2 expands the dataset from 300 to 408 sentences and rebuilds the judge-separated train/validation/test split.

Because Dataset v2 uses a different validation set, its results are **not directly comparable** with Dataset v1 as a controlled one-variable experiment.

## Key Findings

Fine-tuning BGE with triplet loss substantially improved its ability to represent ballot-specific semantic direction.

The three-way formulation also allows the model to distinguish between directional leakage and ordinary neutral feedback.

In the expanded Dataset v2 experiment, only 1 of 33 directional validation sentences was classified as NEUTRAL. Most remaining directional errors were PRO/CON reversals, suggesting that identifying **whether a sentence is directional** is easier than determining **which side it favors**.

## Original BGE Embedding Behavior

Before fine-tuning, BGE assigns high similarity to sentences that are semantically related but imply opposite ballot outcomes.

| Sentence Pair | Cosine Similarity |
|---|---:|
| "Pro wins the debate." / "Con wins the debate." | 0.9404 |
| "Pro has better weighing." / "Con has better weighing." | 0.9861 |
| "Con failed to answer Pro." / "Pro failed to answer Con." | 0.8904 |
| "Pro dropped Con." / "Con dropped Pro." | 0.9086 |
| "Judge votes Pro." / "Judge votes Con." | 0.9551 |

This motivates task-specific fine-tuning: general semantic similarity alone does not reliably encode the directional distinction required for ballot leakage detection.

## Files

- `experiment_summary.csv` — consolidated experiment metrics
- `confusion_matrices/` — confusion matrices for individual experiments
- `figures/` — additional result visualizations

See `../notebooks/` for the implementation and individual experiments.
