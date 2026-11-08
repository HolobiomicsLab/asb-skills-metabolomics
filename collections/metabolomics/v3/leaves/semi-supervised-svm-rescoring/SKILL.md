---
name: semi-supervised-svm-rescoring
description: Use when an unfiltered search-engine PSM list (e.g. XTandem, MaxQuant,
  mzid, tsv) is available as a feature table with target/decoy labels and the goal
  is to increase peptide identifications at a fixed FDR or tighten the FDR threshold.
when_to_use_negative:
- Input already contains post-rescoring q-values/PEPs — re-running the SVM rescoring
  would be redundant.
- Only a single top-ranked PSM per spectrum is needed and no rescoring feature table
  exists — feature generation (MS2PIP, DeepLC, basic features) must be run first.
- Data are not from a target/decoy search design (no decoy identifiers) — the semi-supervised
  training cannot separate positives from negatives.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Percolator
  role: Original semi-supervised SVM rescoring algorithm; ristretto is a lean reimplementation
    of its iterative procedure
  repo: https://github.com/CompOmics/ristretto
- name: ristretto
  role: Fast, dependency-light PSM rescoring engine invoked inside MS²Rescore; takes
    a pandas DataFrame of features, target/decoy labels, and spectrum identifiers
    and returns scores, q-values, and PEPs
  repo: https://github.com/CompOmics/ristretto
- name: MS2Rescore
  role: 'Orchestrating platform that runs the ristretto rescoring step (rescoring_config:
    train_fdr, model) and writes rescored_psms.ms2rescore.tsv'
  repo: https://github.com/compomics/ms2rescore
- name: Mumble
  role: Optional PSM generator that adds candidate Unimod modifications (open modification
    searching) before feature generation and rescoring
  repo: https://github.com/compomics/mumble
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:7f52f6d05231b596d27cf22ef48e6e505f5e9260d5efc03985ae5ff7aaa55879
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/CompOmics/ristretto
  tools:
  - MS2Rescore
  - Mumble
  - Percolator
  - Ristretto
  - ristretto
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# semi-supervised-svm-rescoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Iterative semi-supervised linear-SVM rescoring of peptide-spectrum matches (PSMs), in the Percolator/Ristretto style, that separates correct from incorrect identifications by re-scoring a feature table with target/decoy labels and reporting q-values and PEPs at the PSM, peptide, and protein level.

## When to use

Use when an unfiltered search-engine PSM list (e.g. XTandem, MaxQuant, mzid, tsv) is available as a feature table with target/decoy labels and the goal is to increase peptide identifications at a fixed FDR or tighten the FDR threshold.

## When NOT to use

- Input already contains post-rescoring q-values/PEPs — re-running the SVM rescoring would be redundant.
- Only a single top-ranked PSM per spectrum is needed and no rescoring feature table exists — feature generation (MS2PIP, DeepLC, basic features) must be run first.
- Data are not from a target/decoy search design (no decoy identifiers) — the semi-supervised training cannot separate positives from negatives.

## Inputs

- Unfiltered search-engine PSM list (mzid, tsv, or PSM_utils-readable formats) with rescoring features
- Target/decoy label column derived from the decoy identifier pattern in the identification file
- Spectrum identifier linking each PSM to the original spectra (spectrum_id_pattern, psm_id_pattern)
- Optional: multiple candidate PSMs per spectrum for multi-rank rescoring
- Optional: Mumble-generated candidate PSMs from open-modification-search proposals

## Outputs

- Rescored scores, q-values, and posterior error probabilities (PEPs) at the PSM level
- Peptide- and protein-level q-values and PEPs
- Per-PSM table of rescored scores and q-values (e.g. rescored_psms.ms2rescore.tsv)
- Filtered output: only PSMs up to max_psm_rank_output retained before final FDR calculation

## How to apply

Split PSMs into positives (top-ranked target PSMs passing a 1% training FDR threshold, train_fdr=0.01) and negatives (decoy PSMs identified by the id_decoy_pattern regex, e.g. 'rev_'). Iteratively train a linear SVM (the 'svm' model in ristretto, the semi-supervised procedure of Käll et al. 2007, matching the Percolator/mokapot default): score all PSMs, re-select positives at train_fdr 0.01, retrain until the score distribution converges. Tune the SVM regularization hyperparameter via three-fold cross-validation with nested grid search, training only on the training fold. Apply the final model to every input PSM, filter lower-ranking PSMs before final FDR calculation (max_psm_rank_output, default 1) to ensure correct FDR control, and compute per-PSM q-values from the target-decoy counts over the rescored score distribution. Faster alternative: 'lda', a single-pass Fisher linear discriminant (no iteration).

## Related tools

- **Percolator** (Original semi-supervised SVM rescoring algorithm; ristretto is a lean reimplementation of its iterative procedure) — https://github.com/CompOmics/ristretto
- **ristretto** (Fast, dependency-light PSM rescoring engine invoked inside MS²Rescore; takes a pandas DataFrame of features, target/decoy labels, and spectrum identifiers and returns scores, q-values, and PEPs) — https://github.com/CompOmics/ristretto
- **MS2Rescore** (Orchestrating platform that runs the ristretto rescoring step (rescoring_config: train_fdr, model) and writes rescored_psms.ms2rescore.tsv) — https://github.com/compomics/ms2rescore
- **Mumble** (Optional PSM generator that adds candidate Unimod modifications (open modification searching) before feature generation and rescoring) — https://github.com/compomics/mumble

## Examples

```
ms2rescore --psm-file psms.tsv --spectrum-file spectra.mzML --config '{"rescoring_config":{"train_fdr":0.01,"model":"svm"}}'
```

## Evaluation signals

- Every input PSM receives a finite rescored score.
- All q-values lie within [0, 1] and are monotonically non-decreasing with score rank.
- Decoy-derived FDR control at the reported 0.01 train_fdr threshold is respected (decoy hits at q=0.01 match the target decoy fraction).
- Output retains only PSMs up to max_psm_rank_output (default 1) before final FDR calculation.
- Peptide IDs increase at the same FDR threshold, or a more stringent FDR threshold retains a similar number of peptide IDs.

## Limitations

- ristretto is not as flexible as Mokapot — it exposes only train_fdr and model ('svm' or 'lda'); hyperparameters are tuned once rather than every iteration.
- Training FDR of 0.01 must be respected; PSMs are filtered to max_psm_rank_output before final FDR calculation to ensure correct FDR control.
- Mumble-generated candidate PSMs with a large drop in matched fragment ion fraction (matched_ions_pct) are removed before rescoring.
- Search-engine modification labels must be mapped to ProForma v2 via a modification_mapping, and PSMs must be linked to spectra via spectrum_id_pattern / psm_id_pattern regexes; misconfiguration breaks rescoring.

## Evidence

- [readme] Iterative semi-supervised linear-SVM rescoring of peptide-spectrum matches (PSMs), in the Percolator/Ristretto style, that separates correct from incorrect identifications by re-scoring a feature table with target/decoy labels and reporting q-values and PEPs at the PSM, peptide, and protein level.: [evidence span withheld — anchor status: external_doc]
- [readme] Iteratively train a linear SVM (the 'svm' model in ristretto, the semi-supervised procedure of Käll et al. 2007, matching the Percolator/mokapot default).: [evidence span withheld — anchor status: external_doc]
- [readme] Tune the SVM regularization hyperparameter via three-fold cross-validation with nested grid search, training only on the training fold.: [evidence span withheld — anchor status: external_doc]
- [readme] Faster alternative: 'lda', a single-pass Fisher linear discriminant (no iteration).: [evidence span withheld — anchor status: external_doc]
