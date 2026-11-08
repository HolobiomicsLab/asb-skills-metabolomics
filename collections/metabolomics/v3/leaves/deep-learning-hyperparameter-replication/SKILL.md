---
name: deep-learning-hyperparameter-replication
description: Use when comparing model variants that must differ only in one design
  factor (e.g. single-mode vs. dual-mode training) so observed performance differences
  can be attributed to that factor rather than hyperparameter drift.
when_to_use_negative:
- Input spectra are too few to train from scratch (< ~100,000 spectra); README recommends
  instead fine-tuning the pre-trained model or merging small custom datasets into
  a larger public library.
- The comparison target is a hyperparameter change rather than a training-data change
  — a different sampling algorithm or metadata input would confound attribution.
- Spectra come from a very different chemical domain than the reference training data;
  per-bin loss comparisons to the reference may not transfer.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: ms2deepscore
  role: Provides SettingsMS2Deepscore hyperparameters, train_ms2deepscore_wrapper,
    and MS2DeepScore scoring used to retrain single-ionization-mode models with the
    same settings as the dual-mode reference
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: DEFAULT_FILTERS pipeline cleans spectra before training/evaluation
  repo: https://github.com/matchms/matchms
- name: PyTorch
  role: Backend on which the Siamese model and training loop are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:bafeecf8b3925cb128183b9643e6190f0bf3797cd6956802c5ade5082f0b0dc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - PyTorch
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# deep-learning-hyperparameter-replication

## Summary

Retrain a deep-learning model (here MS2DeepScore 2.0, a Siamese network predicting Tanimoto similarity from MS/MS spectra) under identical hyperparameters as a reference model to isolate the effect of a single design choice, such as training on one vs. both ionization modes.

## When to use

Use when comparing model variants that must differ only in one design factor (e.g. single-mode vs. dual-mode training) so observed performance differences can be attributed to that factor rather than hyperparameter drift.

## When NOT to use

- Input spectra are too few to train from scratch (< ~100,000 spectra); README recommends instead fine-tuning the pre-trained model or merging small custom datasets into a larger public library.
- The comparison target is a hyperparameter change rather than a training-data change — a different sampling algorithm or metadata input would confound attribution.
- Spectra come from a very different chemical domain than the reference training data; per-bin loss comparisons to the reference may not transfer.

## Inputs

- Cleaned reference spectra in positive and/or negative ionization mode (MGF/MSP/MZML etc., via matchms)
- SettingsMS2Deepscore hyperparameter configuration from the reference model
- Tanimoto bin labels for pair sampling and per-bin loss evaluation

## Outputs

- Trained Siamese model checkpoint (.pt) with identical hyperparameters to the reference except the manipulated factor
- Per-Tanimoto-bin loss table for the candidate model on same-mode held-out pairs
- model_comparison.csv containing summary metrics and within-mode loss comparisons vs. the dual-mode reference

## How to apply

Extract the reference model's full hyperparameter set from its SettingsMS2Deepscore class or training pipeline, then train a candidate model that holds every setting fixed while changing only the target factor (e.g. ionisation_mode="positive" instead of "both"). Use the ms2deepscore training wrapper with the same additional_metadata preprocessing (CategoricalToBinary on ionmode, StandardScaler on precursor_mz) and the same pair sampling algorithm balancing pairs over Tanimoto bins. Evaluate on held-out same-mode spectrum pairs by computing losses per Tanimoto bin so the comparison mirrors the reference model's evaluation rather than relying on a single aggregate metric. Write the per-bin loss table and summary metrics to a comparison CSV and verify that within-mode performance of the single-mode model is comparable to the corresponding within-mode losses of the dual-mode reference.

## Related tools

- **ms2deepscore** (Provides SettingsMS2Deepscore hyperparameters, train_ms2deepscore_wrapper, and MS2DeepScore scoring used to retrain single-ionization-mode models with the same settings as the dual-mode reference) — https://github.com/matchms/ms2deepscore
- **matchms** (DEFAULT_FILTERS pipeline cleans spectra before training/evaluation) — https://github.com/matchms/matchms
- **PyTorch** (Backend on which the Siamese model and training loop are implemented)

## Examples

```
settings = SettingsMS2Deepscore(spectrum_file_path="combined_libraries.mgf", additional_metadata=[("CategoricalToBinary", {"metadata_field": "ionmode", "entries_becoming_one": "positive", "entries_becoming_zero": "negative"}), ("StandardScaler", {"metadata_field": "precursor_mz", "mean": 0, "standard_deviation": 1000})], ionisation_mode="positive"); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Positive-only model's within-mode per-bin losses are comparable to the dual-mode model's positive-mode per-bin losses
- Negative-only model shows lower losses at low Tanimoto bins but higher loss in the 0.9-1.0 Tanimoto bin relative to the dual-mode model
- Hyperparameter settings (other than the manipulated factor) match the reference model's SettingsMS2Deepscore configuration
- Per-bin loss table covers the same Tanimoto bins as the reference evaluation
- Summary metrics and per-bin losses written successfully to model_comparison.csv

## Limitations

- Not having an equal number of pairs per Tanimoto bin for each ionization mode can bias per-bin loss comparisons.
- Pair sampling may need re-optimization for new datasets, particularly for smaller training sets, which can make hyperparameter replication unreliable.
- Fragment binning may cause loss of useful information, affecting comparability of learned representations across models.
- Model may not be well-suited for very specific tasks or chemical classes, so within-mode loss equivalence may not generalize to niche domains.

## Evidence

- [results] Within-ionization-mode performance comparable between positive-only and dual-mode models; negative-only losses lower at low Tanimoto bins, higher in 0.9-1.0 bin: [evidence span withheld — anchor status: not_found]
- [readme] Pair sampling tutorial recommends checking and potentially re-optimizing sampling for new datasets, especially for small training sets: [evidence span withheld — anchor status: external_doc]
- [readme] README states > 100,000 spectra are needed for from-scratch training; otherwise merge with public libraries or fine-tune: [evidence span withheld — anchor status: external_doc]
- [readme] Additional metadata preprocessing used in training: CategoricalToBinary on ionmode and StandardScaler on precursor_mz: "additional_metadata=[("CategoricalToBinary", {"metadata_field": "ionmode",
                                                  "entries_becoming_one": "positive","
