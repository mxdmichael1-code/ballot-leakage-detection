# Dataset

This directory contains the sentence-level annotated dataset used for the ballot leakage detection experiments.

## File

`ballot_leakage_candidate_dataset - Candidate Dataset.csv`

Each row represents one sentence extracted from written feedback in a debate judge ballot.

The dataset is designed to study whether an individual feedback sentence reveals information about the judge's ballot outcome, and if so, which side the sentence favors.

## Annotation Framework

Each sentence is annotated along two dimensions: **leakage direction** and **leakage strength**.

### Leakage Direction

| Label | Meaning |
|---|---|
| `PRO` | The sentence provides information favoring a PRO outcome |
| `CON` | The sentence provides information favoring a CON outcome |
| `NEUTRAL` | The sentence does not reveal a clear ballot outcome |

Examples:

- **PRO:** "Con never answers Pro's weighing."
- **CON:** "Pro drops Con's main impact."
- **NEUTRAL:** "Pro should use more evidence."

### Leakage Strength

| Level | Meaning |
|---|---|
| `0` | No meaningful outcome leakage |
| `1` | Directional signal that implicitly suggests an outcome |
| `2` | Explicit or decisive information revealing the outcome |

Examples:

- **Level 0:** "Pro should explain their weighing more clearly."
- **Level 1:** "Con never responds to Pro's weighing."
- **Level 2:** "Pro wins because Con never responds to their weighing."

Direction and strength are annotated separately. A sentence's recorded ballot outcome is not used to determine its leakage direction.

## Dataset Fields

| Field | Description |
|---|---|
| `sentence_id` | Unique identifier for the sentence |
| `ballot_id` | Identifier for the source ballot |
| `judge_id` | Identifier for the judge |
| `actual_ballot` | Recorded ballot outcome (`PRO` or `CON`) |
| `section` | Section of the ballot containing the sentence |
| `sentence` | Sentence extracted from the ballot and used as the primary model input |
| `previous_sentence` | Sentence immediately preceding the target sentence |
| `next_sentence` | Sentence immediately following the target sentence |
| `leakage_strength` | Annotated leakage strength (`0`, `1`, or `2`) |
| `leakage_direction` | Annotated leakage direction (`PRO`, `CON`, or `NEUTRAL`) |
| `annotation_uncertain` | Indicates whether the annotation was considered uncertain |
| `uncertainty_reason` | Explanation for annotation uncertainty, when applicable |

## Modeling Usage

The current experiments use only the `sentence` field as text input to BGE.

`leakage_direction` provides the primary supervision for the embedding experiments:

- `PRO` sentences are trained toward PRO reference statements.
- `CON` sentences are trained toward CON reference statements.
- `NEUTRAL` sentences are used as an explicit third semantic class in the three-way experiments.

`leakage_strength` is retained as an annotation and diagnostic variable but is not directly predicted by the current model.

`actual_ballot`, `section`, `previous_sentence`, and `next_sentence` are not provided to the current embedding model.

## Data Splitting

Model development uses judge-separated training and validation data to reduce the risk of judge-specific writing patterns appearing across both sets.

The held-out test set is not used during model development.

See `../notebooks/` for the experimental implementation and `../results/` for evaluation results.

## Annotation Considerations

The annotations are intentionally conservative. General praise, criticism, or suggestions for improvement are not considered ballot leakage unless the sentence provides meaningful information about the likely outcome.

For example, negative feedback about PRO does not automatically imply a CON direction, and positive feedback about PRO does not automatically imply a PRO direction.

This distinction is important because the task concerns **ballot information leakage rather than sentiment classification**.
