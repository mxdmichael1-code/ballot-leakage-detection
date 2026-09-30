# Dataset

This directory contains the annotated sentence-level dataset used to train and evaluate the ballot leakage detection system.

Each row represents one sentence extracted from written feedback in a debate judge ballot. Sentences are annotated for both the **direction** and **strength** of potential ballot information leakage.

## Annotation Schema

### Leakage Direction

Indicates which ballot outcome, if any, is implied by the sentence.

| Label | Description |
|---|---|
| `PRO` | The sentence provides information favoring a PRO outcome |
| `CON` | The sentence provides information favoring a CON outcome |
| `NEUTRAL` | The sentence does not reveal a clear ballot outcome |

Examples:

- `PRO`: "Con never responds to Pro's weighing."
- `CON`: "Pro drops Con's main impact."
- `NEUTRAL`: "Pro should provide more evidence."

### Leakage Strength

Indicates how strongly the sentence reveals information about the ballot outcome.

| Level | Description |
|---|---|
| `0` | No meaningful outcome leakage |
| `1` | Directional signal that implicitly suggests an outcome |
| `2` | Explicit or decisive information revealing the outcome |

For example:

- **Level 0:** "Pro should explain their weighing more clearly."
- **Level 1:** "Con never responds to Pro's weighing."
- **Level 2:** "Pro wins because Con never responds to their weighing."

Direction and strength are annotated separately.

## Dataset Fields

| Field | Description |
|---|---|
| `sentence_id` | Unique sentence identifier |
| `ballot_id` | Identifier for the source ballot |
| `judge_id` | Anonymized judge identifier |
| `actual_ballot` | Recorded PRO/CON ballot outcome |
| `section` | Ballot section containing the sentence |
| `sentence` | Sentence used as model input |
| `previous_sentence` | Previous sentence in the original feedback |
| `next_sentence` | Following sentence in the original feedback |
| `leakage_strength` | Annotated leakage strength (`0`, `1`, `2`) |
| `leakage_direction` | Annotated direction (`PRO`, `CON`, `NEUTRAL`) |
| `annotation_uncertain` | Whether the annotation was considered uncertain |
| `uncertainty_reason` | Reason for annotation uncertainty, when applicable |

## Dataset Versions

### v1

The original dataset contains **300 sentences from 28 unique ballots and 16 judges**.

| Split | Sentences |
|---|---:|
| Train | 212 |
| Validation | 44 |
| Test | 44 |
| **Total** | **300** |

### v2

The expanded dataset adds **108 manually annotated directional sentences**, producing **408 sentences** in total.

| Split | Sentences |
|---|---:|
| Train | 285 |
| Validation | 61 |
| Test | 62 |
| **Total** | **408** |

Because the dataset was re-split after expansion, results obtained on v1 and v2 validation sets should **not be directly compared as if only the training data changed**.

## Split Strategy

Splits are constructed at the **judge level** rather than randomly at the sentence level.

A judge appearing in one split does not appear in another. This prevents sentences written by the same judge, including potentially similar writing patterns, from leaking across training and evaluation sets.

Ballot and sentence overlap across splits is also checked during dataset construction.

## Model Usage

The current embedding models use only the `sentence` field as text input.

`leakage_direction` is used to construct training triplets and evaluate predictions. Context fields such as `previous_sentence`, `next_sentence`, and `section` are retained for analysis and future experiments but are not included in the current model input.

`leakage_strength` is an annotation and diagnostic variable; the current model is not directly trained to predict leakage strength.
