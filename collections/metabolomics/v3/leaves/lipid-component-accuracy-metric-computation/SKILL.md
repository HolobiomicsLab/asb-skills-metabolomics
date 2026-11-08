---
name: lipid-component-accuracy-metric-computation
description: Use when evaluating LipiDetective transformer predictions on labeled
  test MS2 spectra and you need a finer-grained accuracy metric than whole-species
  match rate, e.g. to show performance when full species predictions are partly wrong.
when_to_use_negative:
- Input is unlabeled mzML prediction data with no ground-truth lipid_species annotations,
  so component accuracy cannot be computed (LipiDetective supports mzML for prediction
  only)
- You need database-style spectrum matching against reference libraries (e.g. Alex123
  fragment matching) rather than neural model evaluation
- Spectra have not been preprocessed consistently with training data (smoothing, baseline
  correction, peak picking, base peak normalization, m/z range 50-1600), making scores
  uncomparable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Trained transformer model whose lipid species predictions are decoded and
    tokenized for component-accuracy scoring
  repo: https://github.com/LipiTUM/lipidetective
- name: h5py
  role: Reads the HDF5 test dataset of labeled MS2 spectra via the H5Dataset loader
- name: PyTorch
  role: Underlying transformer implementation used to generate predictions with greedy
    or beam search decoding
- name: PyTorch Lightning
  role: Runs the evaluation loop with the custom Evaluator that writes per-spectrum
    and overall metrics to test_metrics.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:c26e5f62e5f8ceeb7313b27382d7d65844d2faf34ce7ae30429fbe643cc47087
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
yes_rate: 1.0
n_claims: 9
---

# lipid-component-accuracy-metric-computation

## Summary

Computes per-spectrum lipid component accuracy for LipiDetective predictions by tokenizing predicted and true lipid species names into headgroup, fatty acyl chain, and adduct components and scoring fraction of correctly predicted tokens, complementing strict total (all-tokens-match) prediction accuracy.

## When to use

Use when evaluating LipiDetective transformer predictions on labeled test MS2 spectra and you need a finer-grained accuracy metric than whole-species match rate, e.g. to show performance when full species predictions are partly wrong.

## When NOT to use

- Input is unlabeled mzML prediction data with no ground-truth lipid_species annotations, so component accuracy cannot be computed (LipiDetective supports mzML for prediction only)
- You need database-style spectrum matching against reference libraries (e.g. Alex123 fragment matching) rather than neural model evaluation
- Spectra have not been preprocessed consistently with training data (smoothing, baseline correction, peak picking, base peak normalization, m/z range 50-1600), making scores uncomparable

## Inputs

- Labeled test MS2 spectra in HDF5 format (2xN m/z/intensity arrays with lipid_species, adduct, and precursor metadata attributes)
- Trained LipiDetective transformer model checkpoint
- YAML configuration file specifying HDF5 test file paths and evaluation parameters

## Outputs

- test_metrics.csv with per-spectrum loss, lipid component accuracy (0-1), and total prediction accuracy columns
- Overall component accuracy summary (fraction of correctly predicted tokens across the test set)

## How to apply

Load the labeled test spectra from the HDF5 dataset (each spectrum stored as a 2xN m/z/intensity array with lipid_species, adduct, and precursor metadata attributes) using h5py and the LipiDetective H5Dataset loader. Generate lipid species predictions for each test spectrum with the trained LipiDetective transformer using greedy or beam search decoding. Tokenize both predicted and true lipid species names into components (headgroup, fatty acid side chains, adduct) by regular-expression matching, sorting fatty acids by increasing carbon chain length and then number of double bonds so token order is canonical. Compute lipid component accuracy per spectrum as correctly predicted tokens divided by total tokens, and total prediction accuracy requiring all tokens to match. Aggregate and write per-spectrum and overall metrics, including loss and accuracy columns, to test_metrics.csv via PyTorch Lightning's evaluation loop with the custom Evaluator. Rationale: because LipiDetective composes predictions from relevant lipid components rather than matching overall spectra, component-level scoring reveals performance (reported 60-80% average) that total accuracy obscures.

## Related tools

- **LipiDetective** (Trained transformer model whose lipid species predictions are decoded and tokenized for component-accuracy scoring) — https://github.com/LipiTUM/lipidetective
- **h5py** (Reads the HDF5 test dataset of labeled MS2 spectra via the H5Dataset loader)
- **PyTorch** (Underlying transformer implementation used to generate predictions with greedy or beam search decoding)
- **PyTorch Lightning** (Runs the evaluation loop with the custom Evaluator that writes per-spectrum and overall metrics to test_metrics.csv)

## Examples

```
uv run lipidetective --config config/config_templates/evaluate.yaml
```

## Evaluation signals

- test_metrics.csv exists and contains a lipid component accuracy value in the range 0-1 for every test spectrum
- Overall component accuracy summary is consistent with the fraction of correctly predicted tokens and falls in the reported 60-80% band
- Component accuracy per spectrum is always >= total prediction accuracy per spectrum (all-token match implies all components correct)
- Tokenization is deterministic and canonical: fatty acids sorted by increasing carbon chain length then number of double bonds produce identical token sequences for equivalent species names
- Loss column present per spectrum and overall accuracy summary aggregates match a recomputation from per-spectrum rows

## Limitations

- Component accuracy is much higher and less variable than total prediction accuracy, so it should not be reported as a substitute for whole-species correctness
- Regular-expression tokenization depends on lipid shorthand nomenclature; source nomenclatures were manually adjusted to the updated lipid shorthand notation, and nonconforming names will tokenize incorrectly
- Average component accuracy around 60-80% means on average only roughly half or more of tokens are correct even when the full species prediction is inaccurate — the metric does not guarantee correct molecular lipid species assignment

## Evidence

- [results] LipiDetective's lipid component accuracy is less variable and much higher than its total prediction accuracy, averaging between 60% and 80%: [evidence span withheld — anchor status: not_found]
- [intro] it does not have to rely on matching the overall spectra but can identify relevant lipid components from which it composes the final prediction: "it does not have to rely on matching the overall spectra but can identify relevant lipid components from which it composes the final prediction."
- [methods] all spectra were saved in a Hierarchical Data Format 5 (HDF5) file: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [readme] Supports training with HDF5 files and prediction using mzML files: [evidence span withheld — anchor status: external_doc]
- [methods] The varying nomenclatures in the different data sources were manually adjusted to match the updated lipid shorthand notation: "The varying nomenclatures in the different data sources were manually adjusted to match the updated lipid shorthand notation"
