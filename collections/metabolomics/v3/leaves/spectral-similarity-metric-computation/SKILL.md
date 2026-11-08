---
name: spectral-similarity-metric-computation
description: Use when comparing a predicted or library MS/MS spectrum against an experimental
  spectrum to quantify agreement — e.g. benchmarking in silico spectral prediction
  or ranking candidate compound matches.
when_to_use_negative:
- The input is a single unlabeled spectrum with no reference spectrum to compare against
  — similarity metrics require a pair of spectra.
- Peaks are matched by retention-time or MS1 features only, not by m/z peak lists
  — this skill scores MS/MS peak agreement, not chromatographic similarity.
edam_operation: http://edamontology.org/operation_0405
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CFM-ID 4.0
  role: Produces the predicted ESI-MS/MS spectra that are scored against experimental
    spectra with the Dice/Precision/Recall/Dot Product metrics
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: CFM-ID 2.0
  role: Baseline predictor whose spectra are scored with the same metrics to quantify
    improvement
- name: CFM-ID 3.0
  role: Baseline predictor whose spectra are scored with the same metrics to quantify
    improvement
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:016093df54b539ccd24df12f07300740bfdb3377c556f788eab9efe1cb2d0220
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 2.0
  - CFM-ID 3.0
  - CFM-ID 4.0
  - Docker
  - Metlin
  edam_operation: http://edamontology.org/operation_0405
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-similarity-metric-computation

## Summary

Computes spectral similarity metrics (Dice coefficient, Precision, Recall, and re-weighted Stein's Dot Product) between predicted and experimental MS/MS spectra, using the CFM-ID 4.0 paper's peak-matching rule of m/z difference < 0.01 Da or 10 ppm (whichever first) to score ESI-MS/MS spectral prediction quality.

## When to use

Use when comparing a predicted or library MS/MS spectrum against an experimental spectrum to quantify agreement — e.g. benchmarking in silico spectral prediction or ranking candidate compound matches.

## When NOT to use

- The input is a single unlabeled spectrum with no reference spectrum to compare against — similarity metrics require a pair of spectra.
- Peaks are matched by retention-time or MS1 features only, not by m/z peak lists — this skill scores MS/MS peak agreement, not chromatographic similarity.

## Inputs

- Predicted MS/MS spectrum (peak list of m/z and intensity pairs, e.g. CFM-ID output at 10, 20, 40 eV)
- Experimental QTOF ESI-MS/MS spectrum (peak list from Metlin or equivalent, per ionization mode and collision energy)

## Outputs

- Per-spectrum metric values: Dice coefficient, Precision, Recall, re-weighted Stein's Dot Product
- Per-energy-level and averaged metric tables (e.g. CSV) suitable for t-test comparison across model versions

## How to apply

For each compound and collision energy (e.g. 10, 20, 40 eV), align the predicted spectrum against the experimental QTOF ESI-MS/MS peak list by treating peaks as matched when their m/z difference is < 0.01 Da or < 10 ppm, whichever bound is reached first. From the matched/unmatched peak sets, compute the Dice coefficient (peak presence agreement, 2·|common|/(|pred|+|exp|)), Precision (fraction of predicted peaks matched), Recall (fraction of experimental peaks matched), and Stein's Dot Product on the intensity vectors, re-weighted to penalize large unmatched peaks. Apply the same post-processing to predicted spectra first: iteratively retain the most intense peaks until the summed intensity reaches 80% of the total or 30 peaks are selected, so that metrics are computed on comparable peak lists. Average the metrics across all energy levels and folds (e.g. under 10-fold cross-validation) and report per-energy-level and averaged values, so improvements between model versions (like CFM-ID 4.0 vs 2.0/3.0) can be tested with a two-sided t-test.

## Related tools

- **CFM-ID 4.0** (Produces the predicted ESI-MS/MS spectra that are scored against experimental spectra with the Dice/Precision/Recall/Dot Product metrics) — https://bitbucket.org/wishartlab/cfm-id-code/
- **CFM-ID 2.0** (Baseline predictor whose spectra are scored with the same metrics to quantify improvement)
- **CFM-ID 3.0** (Baseline predictor whose spectra are scored with the same metrics to quantify improvement)

## Examples

```
python score_spectra.py --pred pred_mh.tsv --exp exp_mh.tsv --mz_tol 0.01 --ppm_tol 10 --metrics dice,precision,recall,dot_product
```

## Evaluation signals

- Dice coefficient should be in [0, 1]; identical peak sets must yield Dice = 1.0
- Precision and Recall should each be in [0, 1]; Precision ≤ 1 and Recall ≤ 1 must hold for every spectrum
- Peak matching must use the m/z tolerance < 0.01 Da or 10 ppm, whichever is smaller — verify a matched peak satisfies at least one of the two bounds
- Re-weighted Stein's Dot Product should be ≥ 0 and maximal (≈ 1 for identical, normalized spectra)
- Averaged Dice and Dot Product improvements of CFM-ID 4.0 over CFM-ID 2.0/3.0 should be positive and directionally consistent with published values (≈ +31.8% Dice on [M+H]+, ≈ +8% Dice on [M-H]−, ≈ +26.7% Dot Product on [M+H]+, ≈ +20.6% Dot Product on [M-H]−)

## Limitations

- Peak-matching tolerance is fixed (< 0.01 Da or 10 ppm); spectra from instruments with lower mass accuracy may be mismatched
- Metrics are computed on QTOF ESI-MS/MS spectra; adapting to non-QTOF instruments is listed as future work in the paper
- Dot Product is sensitive to intensity scaling; the re-weighted variant must be applied consistently across compared spectra

## Evidence

- [methods] peak matches defined as m/z difference < 0.01 Da or 10 ppm: [evidence span withheld — anchor status: not_found]
- [methods] post-processing step that iteratively keeps the most intense peaks until the summed intensity reaches 80% of the total or 30 peaks are selected: [evidence span withheld — anchor status: not_found]
- [methods] Score each predicted spectrum against the experimental spectrum using peak matches defined as m/z difference < 0.01 Da or 10 ppm (whichever first): [evidence span withheld — anchor status: not_found]
