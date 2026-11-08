---
name: sequence-prediction-evaluation
description: 'Use when you have a sequence-generating model''s predicted labels (e.g.
  lipid species strings from tandem MS2 spectra) alongside ground truth, and need
  accuracy beyond exact-match: fraction of correctly predicted tokens per instance
  and overall.'
when_to_use_negative:
- The downstream question only needs whole-spectrum exact-match species identification
  statistics — in that case simple total accuracy suffices and tokenization adds no
  signal.
- Predictions are regression outputs or scalar scores rather than discrete token sequences
  — component-token accuracy is undefined.
- The label nomenclature has not been harmonized to a single shorthand (e.g. mixed
  lipid dialects); the article notes varying nomenclatures in different data sources
  had to be manually adjusted to the updated lipid shorthand notation before tokenization
  is meaningful.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3520
tools:
- name: LipiDetective
  role: Transformer model whose lipid species predictions are decoded and tokenized
    for component-accuracy evaluation
  repo: https://github.com/LipiTUM/lipidetective
- name: h5py
  role: Loads the labeled test spectra stored in HDF5 format (2xN m/z/intensity arrays
    with lipid_species, adduct, precursor attributes)
- name: PyTorch
  role: Backend of the LipiDetective transformer whose greedy/beam-search decoding
    outputs are evaluated
- name: PyTorch Lightning
  role: Runs the evaluation loop with a custom Evaluator that aggregates component
    accuracy and writes test_metrics.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:f71fa437f26a9ff22ca7c2c052c77934b32ab2ed69e3b0cfa3f78ac8541b060e
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# sequence-prediction-evaluation

## Summary

Evaluates a transformer's sequence predictions by tokenizing predicted and true lipid species names into components (headgroup, fatty acyl chains, adduct) and reporting both per-spectrum component accuracy and all-token total accuracy, so that partial credit on sub-structures is visible even when the full species prediction is wrong.

## When to use

Use when you have a sequence-generating model's predicted labels (e.g. lipid species strings from tandem MS2 spectra) alongside ground truth, and need accuracy beyond exact-match: fraction of correctly predicted tokens per instance and overall.

## When NOT to use

- The downstream question only needs whole-spectrum exact-match species identification statistics — in that case simple total accuracy suffices and tokenization adds no signal.
- Predictions are regression outputs or scalar scores rather than discrete token sequences — component-token accuracy is undefined.
- The label nomenclature has not been harmonized to a single shorthand (e.g. mixed lipid dialects); the article notes varying nomenclatures in different data sources had to be manually adjusted to the updated lipid shorthand notation before tokenization is meaningful.

## Inputs

- HDF5 (.h5) file of labeled test MS2 spectra stored as 2xN m/z/intensity arrays with lipid_species, adduct, and precursor metadata attributes
- Trained LipiDetective transformer checkpoint
- Predicted and true lipid species name strings in updated lipid shorthand notation (e.g. headgroup plus fatty acyl chains plus adduct tokens)

## Outputs

- test_metrics.csv with per-spectrum lipid component accuracy and total prediction accuracy (values in 0-1), plus loss and accuracy columns
- Overall component accuracy summary equal to the fraction of correctly predicted tokens across all test spectra

## How to apply

Generate lipid species predictions for each labeled test MS2 spectrum with the trained LipiDetective transformer using greedy or beam search decoding, reading spectra from the HDF5 dataset (2xN m/z/intensity arrays with lipid_species, adduct, and precursor metadata) via the LipiDetective H5Dataset loader. Tokenize both predicted and true lipid species names into components — headgroup, fatty acid side chains, adduct — by regular-expression matching, sorting fatty acids by increasing carbon chain length and then number of double bonds. Compute lipid component accuracy per spectrum as correctly predicted tokens divided by total tokens, and total prediction accuracy requiring all tokens to match. Aggregate scores with PyTorch Lightning's evaluation loop and a custom Evaluator, writing per-spectrum and overall metrics to test_metrics.csv. The article reports component accuracy averaging 60-80% versus much lower and more variable total accuracy, which is the key rationale: token-level scoring reveals that on average more than half of the lipid component tokens are identified correctly even when the full species prediction is inaccurate.

## Related tools

- **LipiDetective** (Transformer model whose lipid species predictions are decoded and tokenized for component-accuracy evaluation) — https://github.com/LipiTUM/lipidetective
- **h5py** (Loads the labeled test spectra stored in HDF5 format (2xN m/z/intensity arrays with lipid_species, adduct, precursor attributes))
- **PyTorch** (Backend of the LipiDetective transformer whose greedy/beam-search decoding outputs are evaluated)
- **PyTorch Lightning** (Runs the evaluation loop with a custom Evaluator that aggregates component accuracy and writes test_metrics.csv)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- test_metrics.csv exists and every per-spectrum lipid component accuracy value lies in [0, 1]
- Overall component accuracy equals the aggregate fraction of correctly predicted tokens; the article's reference regime is 60-80% (more than half of tokens correct on average) even when total accuracy is much lower and more variable
- Component accuracy should be less variable and substantially higher than total prediction accuracy across spectra — a sanity invariant of token-level partial-credit scoring
- Token counts are consistent: fatty acids sorted by increasing carbon chain length then number of double bonds, and headgroup/fatty acid/adduct tokens exhaustively cover both predicted and true strings
- Metrics columns (loss, accuracy, lipid component accuracy) are populated for every test spectrum with no NaN rows

## Limitations

- Component accuracy is lenient: a prediction with most tokens right but one wrong fatty acyl chain counts highly while still being an incorrect species — total accuracy is needed alongside it.
- Requires manual/regex harmonization of lipid nomenclature to the updated lipid shorthand notation; differing nomenclatures in source data would break tokenization.
- The interpretation (component accuracy 60-80%) is tied to LipiDetective's training set of 268,720 spectra and its test datasets (PNNL, HCE, IOBA-NHC, MITOMICS, Thermo, SRM 1950); absolute values may not transfer to other models or corpora.

## Evidence

- [results] component accuracy definition and 60-80% finding: [evidence span withheld — anchor status: not_found]
- [intro] tokenization rationale from intro: "it does not have to rely on matching the overall spectra but can identify relevant lipid components from which it composes the final prediction."
- [methods] evaluation loop and output file: "PyTorch Lightning [27] was used for multi-GPU training"
- [methods] nomenclature harmonization prerequisite: "The varying nomenclatures in the different data sources were manually adjusted to match the updated lipid shorthand notation"
- [readme] HDF5 input format from README: [evidence span withheld — anchor status: external_doc]
