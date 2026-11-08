---
name: token-level-accuracy-evaluation
description: Use when scoring lipid species predictions against labels for held-out
  MS2 spectra and you need both strict whole-sequence accuracy and token/component-level
  diagnostics rather than a single overall match rate.
when_to_use_negative:
- When the model output is a single class label rather than a multi-token lipid shorthand
  sequence — a plain accuracy metric suffices and token decomposition is meaningless.
- When evaluating spectral similarity (e.g. cosine/matching scores between spectra)
  rather than identity of the decoded lipid notation string.
- When the input spectra are raw profile data that has not been preprocessed (smoothing,
  baseline correction, peak picking, base peak normalization, m/z 50–1600 trimming)
  — token evaluation presupposes preprocessed peak lists.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Produces the token-sequence predictions being scored; its test workflow writes
    the metrics, predictions, per-lipid metrics, and confusion matrix CSVs
  repo: https://github.com/LipiTUM/lipidetective
- name: PyTorch
  role: Underlying transformer implementation whose autoregressive decoding yields
    the per-token predictions evaluated here
- name: PyTorch Lightning
  role: Runs the test loop that logs per-step loss and accuracy to test_metrics.csv
- name: h5py
  role: Reads the HDF5 test file containing labeled held-out spectra used as evaluation
    ground truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:7ac6933d4ca63780cb07ef0e089d3e61f9cd78537aeb45eca4d47e39ad7dedbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - h5py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# token-level-accuracy-evaluation

## Summary

Evaluates transformer lipid-identification predictions by decomposing the lipid shorthand notation into tokens (headgroup, fatty acid side chains, adduct) and scoring exact token-sequence matches, plus per-token component accuracy, per-lipid-class precision/recall/F1, and a confusion matrix.

## When to use

Use when scoring lipid species predictions against labels for held-out MS2 spectra and you need both strict whole-sequence accuracy and token/component-level diagnostics rather than a single overall match rate.

## When NOT to use

- When the model output is a single class label rather than a multi-token lipid shorthand sequence — a plain accuracy metric suffices and token decomposition is meaningless.
- When evaluating spectral similarity (e.g. cosine/matching scores between spectra) rather than identity of the decoded lipid notation string.
- When the input spectra are raw profile data that has not been preprocessed (smoothing, baseline correction, peak picking, base peak normalization, m/z 50–1600 trimming) — token evaluation presupposes preprocessed peak lists.

## Inputs

- test_predictions.csv: one prediction-vs-label row per test spectrum (autoregressively decoded lipid shorthand token sequences)
- Labeled MS2 test spectra from the HDF5 test file (2xN m/z/intensity arrays with lipid_species, adduct, precursor, polarity attributes)

## Outputs

- test_metrics.csv: per-step loss and total prediction accuracy
- test_predictions.csv: prediction vs label per test spectrum
- test_lipid_metrics.csv: per-lipid precision/recall/F1
- confusion_matrix_test.csv: lipid-class confusion matrix
- scalar total prediction accuracy and lipid component accuracy

## How to apply

Load the predicted and labeled token sequences for each test spectrum and compute total prediction accuracy by exact token-sequence match: a prediction is counted correct only if all tokens — headgroup, fatty acid side chains, and adduct — match the label ('Only if all tokens match will the prediction be classified as correct'). Additionally compute lipid component accuracy as the fraction of correct tokens over total tokens, which distinguishes near-misses (e.g. correct headgroup but wrong chain) from complete failures. Aggregate per-lipid precision, recall, and F1, and build a confusion matrix over lipid classes to reveal systematic confusions between related species. Write results as CSV files: test_metrics.csv (loss and accuracy per step), test_predictions.csv (one prediction-vs-label row per spectrum), test_lipid_metrics.csv, and confusion_matrix_test.csv. Rationale: token-level scoring matches LipiDetective's autoregressive shorthand-notation decoding, so partial correctness at the component level remains measurable even when the strict whole-sequence criterion fails — important when generalizing to lipid species the model has never seen.

## Related tools

- **LipiDetective** (Produces the token-sequence predictions being scored; its test workflow writes the metrics, predictions, per-lipid metrics, and confusion matrix CSVs) — https://github.com/LipiTUM/lipidetective
- **PyTorch** (Underlying transformer implementation whose autoregressive decoding yields the per-token predictions evaluated here)
- **PyTorch Lightning** (Runs the test loop that logs per-step loss and accuracy to test_metrics.csv)
- **h5py** (Reads the HDF5 test file containing labeled held-out spectra used as evaluation ground truth)

## Examples

```
uv run lipidetective --config config/test_config.yaml  # config: workflow.test: True, workflow.load_model: True, test.confidence_score: True, batch 512
```

## Evaluation signals

- test_metrics.csv exists and reports total prediction accuracy computed by exact token-sequence match (all tokens including adduct must match).
- test_predictions.csv contains exactly one prediction-vs-label row per test spectrum.
- test_lipid_metrics.csv reports precision, recall, and F1 for each lipid species/class.
- confusion_matrix_test.csv rows/columns sum consistently with the number of predictions per class.
- Lipid component accuracy (correct tokens / total tokens) is ≥ total prediction accuracy, since partial token matches count in the former but not the latter.

## Limitations

- Strict whole-sequence accuracy is demanding: reported total prediction accuracy on held-out validation splits ranged from 29.7% to 50.8% (low of 9.7% on a 10-spectra split), so small test splits yield highly variable estimates.
- Token-level accuracy can mask additive/mislocalization errors: a wrong fatty-acid chain still counts as partially correct under component accuracy.
- Blank/no-lipid spectra are scored on the end-of-sequence token; spectra without target lipid species must be handled as a distinct prediction outcome.
- Peaks relevant for prediction, such as the 224.0 m/z fragment, lack annotations in the Alex123 database, which can cause systematic mispredictions that surface as recurring confusions.

## Evidence

- [methods] Only if all tokens match will the prediction be classified as correct: "Only if all tokens match will the prediction be classified as correct"
- [results] Total prediction accuracy on held-out validation splits ranged from 29.7% to 50.8%: [evidence span withheld — anchor status: not_found]
- [abstract] The model can even correctly predict lipid species for which it has never seen a spectrum before: "The model can even correctly predict lipid species for which it has never seen a spectrum before as it is able to generalize the learned lipid fragmentation patterns"
- [results] 2002 MS2 spectra from blank measurements that do not show any target lipid species were added: "2002 MS2 spectra from blank measurements that do not show any target lipid species were added."
- [readme] LipiDetective can be executed by running the following command: "uv run lipidetective --config path/to/config.yaml"
