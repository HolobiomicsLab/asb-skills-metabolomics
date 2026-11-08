---
name: ground-truth-formula-matching
description: Use when you have SIRIUS/ZODIAC annotation results plus a ground-truth
  formula file (spectral library search + manual validation) and need per-dataset
  accuracy, incorrect-annotation rates, or error-reduction factors.
when_to_use_negative:
- Annotating a single query spectrum de novo without a validated ground truth — this
  skill evaluates annotation quality, it does not produce new formula assignments
- Datasets whose compounds lack spectral library hits or manual validation, so no
  trusted ground-truth formulas can be established for matching
- Structure-level (CSI:FingerID) identification benchmarking — this skill operates
  only at the molecular formula level
tools:
- name: SIRIUS
  role: Enumerates molecular formula candidates and provides the top-ranked (baseline)
    formula annotation per compound to be matched against the ground truth
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: ZODIAC
  role: Reranks the top 50 SIRIUS formula candidates using Bayesian statistics via
    Gibbs sampling; its top-ranked formula is matched against the same ground truth
    to measure error reduction
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: OpenMS
  role: Modified OpenMS 2.4.0 used to process input mzML/mzXML files and discard low-quality
    MS/MS spectra before annotation
  repo: https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:cbd9ceaad0868b989630c919c72d5f80c20cb815f0f3bc672b7b9f562d80fe39
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - OpenMS
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ground-truth-formula-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill for molecular formula annotation: match top-ranked formulas assigned by SIRIUS (candidate enumeration) and ZODIAC (Gibbs-sampling reranking of the top 50 SIRIUS candidates) against a manually validated ground truth across the dendroides, NIST1950, tomato, diatoms and mice stool LC-MS/MS datasets, then compute per-dataset accuracy and error rates.

## When to use

Use when you have SIRIUS/ZODIAC annotation results plus a ground-truth formula file (spectral library search + manual validation) and need per-dataset accuracy, incorrect-annotation rates, or error-reduction factors.

## When NOT to use

- Annotating a single query spectrum de novo without a validated ground truth — this skill evaluates annotation quality, it does not produce new formula assignments
- Datasets whose compounds lack spectral library hits or manual validation, so no trusted ground-truth formulas can be established for matching
- Structure-level (CSI:FingerID) identification benchmarking — this skill operates only at the molecular formula level

## Inputs

- SIRIUS annotation results (top-ranked molecular formula per compound) for each dataset
- ZODIAC annotation results (reranked top-50 SIRIUS candidates per compound) for each dataset
- Ground-truth molecular formula files established by spectral library search and manual validation
- MassIVE dataset accessions: dendroides MSV000080502, NIST1950 MSV000081364, plus tomato, diatoms, mice stool datasets
- Preprocessed mzML/mzXML files (OpenMS, low-quality MS/MS spectra discarded) if rerunning annotation

## Outputs

- CSV table (5 datasets x 2 methods) with per-dataset formula-annotation accuracy and incorrect-annotation rates
- Per-dataset error-reduction factor (ratio of SIRIUS to ZODIAC incorrect-annotation rates)
- Per-compound correct/incorrect match status against the ground truth

## How to apply

Download the deposited SIRIUS and ZODIAC annotation results and ground-truth formula files from https://bio.informatik.uni-jena.de/data/. For each compound in each dataset, extract the top-ranked molecular formula from SIRIUS (which enumerates formula candidates) and from ZODIAC (which reranks the top 50 SIRIUS candidates via Bayesian statistics with Gibbs sampling, using ten parallel chains, burn-in of 1,000 epochs and sampling of 2,000 epochs). Match each top-ranked formula exactly against the ground truth established by spectral library search (e.g. NIST17 library) and manual validation for the five MassIVE datasets (dendroides MSV000080502, NIST1950 MSV000081364, tomato, diatoms, mice stool). Compute per-dataset accuracy as the fraction of correct top-ranked formulas for each method, then derive the error-reduction factor as the ratio of SIRIUS to ZODIAC incorrect-annotation rates (16.2-fold on dendroides, 197 evaluable compounds). Write the 5-datasets x 2-methods table with accuracy and error-rate columns to CSV and check it reproduces the published values before reuse.

## Related tools

- **SIRIUS** (Enumerates molecular formula candidates and provides the top-ranked (baseline) formula annotation per compound to be matched against the ground truth) — https://bio.informatik.uni-jena.de/software/sirius/
- **ZODIAC** (Reranks the top 50 SIRIUS formula candidates using Bayesian statistics via Gibbs sampling; its top-ranked formula is matched against the same ground truth to measure error reduction) — https://bio.informatik.uni-jena.de/software/sirius/
- **OpenMS** (Modified OpenMS 2.4.0 used to process input mzML/mzXML files and discard low-quality MS/MS spectra before annotation) — https://github.com/marcus-ludwig/OpenMS/tree/release_2.4.0_modifications_ZODIAC_workflow

## Examples

```
python -c "print('dendroides error-reduction factor: %.2f' % ((1-0.5076)/(1-0.9695)))"
```

## Evaluation signals

- dendroides dataset (197 evaluable compounds): SIRIUS ~50.76% correct formulas, ZODIAC (without anchors) ~96.95% correct
- dendroides error-reduction factor reproduces ~16.17-fold (ratio of SIRIUS to ZODIAC incorrect-annotation rates)
- NIST1950 error rates ~8.51% (SIRIUS) vs 5.32% (ZODIAC); tomato ~4.44% vs 2.22%; mice stool ~30.23% vs 20.93%
- diatoms incorrect-annotation rate roughly halved by ZODIAC from ~12.90%
- Every evaluated compound's top-ranked formula must exactly match a ground-truth formula string from the spectral-library/manual-validation set; unmatched compounds are excluded from the evaluable denominator

## Limitations

- SIRIUS 4's formula-annotation performance drops substantially for compounds above 500 Da (dendroides compounds are mostly above 700 Dalton), inflating baseline error rates
- Library-hit anchors contribute only a small effect to molecular formula annotations, so anchor-based ZODIAC runs differ little from unanchored ones
- Ground truth depends on spectral library search plus manual validation; compounds with novel formulas (e.g. C24H47BrNO8P) absent from PubChem/ChemSpider cannot be cross-checked against structure databases
- Reproducibility of the upstream preprocessing requires a modified OpenMS 2.4.0 plus a patch file, not stock OpenMS

## Evidence
