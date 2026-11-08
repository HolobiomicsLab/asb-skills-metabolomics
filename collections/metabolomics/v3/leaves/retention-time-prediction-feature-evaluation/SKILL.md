---
name: retention-time-prediction-feature-evaluation
description: Decision support for choosing or validating a retention time predictor
  (e.g. DeepLC, Prosit-derived RT, RTPredict) whose ΔRT error will serve as a rescoring
  feature in Percolator-style PSM rescoring.
when_to_use_negative:
- The analysis goal is evaluating spectral (MS²PIP/Prosit fragment-intensity) or binding-affinity
  features rather than retention time features
- Retention times of the input PSMs are unavailable or cannot be calibrated per raw
  file, so ΔRT cannot be computed reliably
- Input is already a final scored PSM table with no access to the underlying predictor,
  making feature-weight attribution impossible
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MS2Rescore
  role: Provides configurable RT feature generation (deeplc generator) with per-raw-file
    RT calibration for ΔRT rescoring features
  repo: https://github.com/compomics/ms2rescore
- name: DeepRescore
  role: Benchmark showing ΔRT consistently among the highest-weight rescoring features
    in DDA immunopeptidomics
- name: MHCquant
  role: 'Counter-example: its RTPredict-based ΔRT was ineffective, illustrating the
    predictor-accuracy dependency'
- name: Percolator
  role: Semi-supervised rescoring engine whose learned feature weights are inspected
    to judge ΔRT informativeness
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:9e783985eed59f3e511f6425b15f3f857e523c11e5f073056687cfdc85fd4ade
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - DeepRescore
  - MHCquant
  - MS2Rescore
  - MSBooster
  - Percolator
  - inSPIRE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# retention-time-prediction-feature-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the accuracy of a retention time predictor and its ΔRT feature weight before adopting ΔRT as a PSM rescoring feature, because predictor quality — not the feature concept — determines whether ΔRT improves immunopeptide identification.

## When to use

Decision support for choosing or validating a retention time predictor (e.g. DeepLC, Prosit-derived RT, RTPredict) whose ΔRT error will serve as a rescoring feature in Percolator-style PSM rescoring.

## When NOT to use

- The analysis goal is evaluating spectral (MS²PIP/Prosit fragment-intensity) or binding-affinity features rather than retention time features
- Retention times of the input PSMs are unavailable or cannot be calibrated per raw file, so ΔRT cannot be computed reliably
- Input is already a final scored PSM table with no access to the underlying predictor, making feature-weight attribution impossible

## Inputs

- Search-engine PSM identifications (e.g. mzIdentML/TSV) with observed retention times for target and decoy peptide-spectrum matches
- Raw spectra linked to PSMs via configurable spectrum_id_pattern / psm_id_pattern regex extraction
- A candidate retention time predictor (e.g. DeepLC in MS²Rescore's feature_generators, RTPredict as used by MHCquant, Prosit-derived RT)

## Outputs

- Per-PSM ΔRT (predicted minus calibrated observed retention time) feature column
- Trained rescoring model feature weights showing the relative importance of ΔRT
- PSM counts at a given FDR threshold (e.g. 1%) with versus without the ΔRT feature

## How to apply

Before adding ΔRT to a rescoring feature set, generate predicted retention times for the identified immunopeptides and assess prediction error against the observed LC elution order, ideally calibrating per raw file as MS²Rescore's RT predictor does. Train the semi-supervised rescoring model (Percolator-style; e.g. ristretto with train_fdr=0.01 and the default svm model) with ΔRT included and inspect the learned feature weights: DeepRescore found ΔRT consistently among the highest-weight features, whereas MHCquant concluded ΔRT was ineffective because it relied on the RTPredict predictor (published more than 15 years ago) that provides less accurate predictions. If ΔRT receives low weight and fails to increase identifications at a given FDR, attribute the failure to the predictor and swap in a modern deep-learning RT predictor (e.g. DeepLC, as exposed via MS²Rescore's feature_generators configuration with the 'deeplc' generator) before declaring ΔRT uninformative. The general lesson from the review is that highly accurate predicted peptidoform properties — fragment ion intensities, retention time, and collisional cross section — are required for informative rescoring features.

## Related tools

- **MS2Rescore** (Provides configurable RT feature generation (deeplc generator) with per-raw-file RT calibration for ΔRT rescoring features) — https://github.com/compomics/ms2rescore
- **DeepRescore** (Benchmark showing ΔRT consistently among the highest-weight rescoring features in DDA immunopeptidomics)
- **MHCquant** (Counter-example: its RTPredict-based ΔRT was ineffective, illustrating the predictor-accuracy dependency)
- **Percolator** (Semi-supervised rescoring engine whose learned feature weights are inspected to judge ΔRT informativeness)

## Examples

```
ms2rescore config.toml psm_list.tsv  # with feature_generators = { basic = {}, ms2pip = { model = "HCD" }, ms2 = {}, deeplc = {} }
```

## Evaluation signals

- Feature weights from the trained Percolator/ristretto model show ΔRT ranking near the top of the weight distribution when a modern predictor is used (DeepRescore benchmark expectation)
- Identified PSM counts at a fixed FDR threshold (e.g. 1%) increase when ΔRT is added, versus no gain with an outdated predictor like RTPredict
- Retention time predictions are calibrated per raw file independently before ΔRT computation
- Predicted-versus-observed RT error on held-out peptides is low enough to order peptides correctly within a run
- Target/decoy labels and spectrum identifiers are correctly joined to the rescoring feature table before model training

## Limitations

- A negative ΔRT result with one predictor does not generalize — MHCquant's conclusion with RTPredict contradicted DeepRescore's results, so conclusions are predictor-specific
- Legacy RT predictors (e.g. RTPredict, published more than 15 years ago) provide less accurate predictions and yield uninformative ΔRT features
- Percolator is referenced as an external tool dependency but no version is specified in the reviewed material

## Evidence

- [full_text] RTPredict inaccuracy: [evidence span withheld — anchor status: not_found]
- [full_text] DeepRescore ΔRT weight: [evidence span withheld — anchor status: not_found]
- [readme] ristretto rescoring input: [evidence span withheld — anchor status: external_doc]
