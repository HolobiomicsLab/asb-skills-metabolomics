---
name: plasma-cohort-multimode-comparison
description: Use when reproducing or extending metabolite-annotation accuracy comparisons
  between longitudinal-fragment-profile and cross-sectional entropy matching on human
  plasma LC-MS/MS data in positive/negative, apo/acid modes.
when_to_use_negative:
- Input is raw vendor LC-MS/MS data rather than already-annotated candidate-ranking
  tables — run the upstream 001-007.ipynb processing (spectrum filtering, invalid
  fragment removal, CQMU construction) first
- The cohort is not human plasma or the acquisition modes differ from CP/CN/AP/AN
  — the 0.5 and 0.75 thresholds were ROC-derived on this cohort and the CQMD isomer
  database, and may not transfer
- Data were not acquired at multiple collision energies — longitudinal fragment profiles
  require MS/MS at distinct CEs; single-CE spectra only support cross-sectional matching
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: Optimized longitudinal fragment-profile matcher applied at the 0.5 ROC-derived
    threshold to rank candidate metabolites per mode
- name: 001-007.ipynb
  role: Jupyter notebook series producing the cohort annotation tables and running
    isomer similarity calculations through metabolite identification
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: mzbatch.py
  role: Mass-spectrometry data processing library used for spectrum extraction, filtering,
    and data calculation steps feeding the cohort comparison
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') in which the reproduction and per-mode
    metric computation are executed
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:5a9d89f349964e467088b61b49f65eb1e327fc722463bf4bd08f02b4c1034b10
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# plasma-cohort-multimode-comparison

## Summary

Benchmarks longitudinal MS/MS fragment profiling (Jaccard similarity, 0.5 threshold) against cross-sectional optimal-entropy matching (0.75 threshold) on the CKD human plasma cohort across CP, CN, AP, and AN acquisition modes, quantifying annotation accuracy gains and FDR reductions.

## When to use

Use when reproducing or extending metabolite-annotation accuracy comparisons between longitudinal-fragment-profile and cross-sectional entropy matching on human plasma LC-MS/MS data in positive/negative, apo/acid modes.

## When NOT to use

- Input is raw vendor LC-MS/MS data rather than already-annotated candidate-ranking tables — run the upstream 001-007.ipynb processing (spectrum filtering, invalid fragment removal, CQMU construction) first
- The cohort is not human plasma or the acquisition modes differ from CP/CN/AP/AN — the 0.5 and 0.75 thresholds were ROC-derived on this cohort and the CQMD isomer database, and may not transfer
- Data were not acquired at multiple collision energies — longitudinal fragment profiles require MS/MS at distinct CEs; single-CE spectra only support cross-sectional matching

## Inputs

- Supplementary Excel tables S8–S13 with CKD plasma cohort annotation results (CP, CN, AP, AN modes)
- CQMU/CQMD in-house MS/MS reference database entries for candidate metabolites
- Multi-collision-energy MS/MS spectra (13 distinct CEs, stepped NCE 20/40/60%) from human plasma samples

## Outputs

- figure5cd_reproduction.csv with per-mode accuracy, improvement, and FDR values
- Per-mode counts of correct top-ranked annotations (expected: 518 longitudinal vs 485 entropy)
- Per-mode FDR comparison table across CP/CN/AP/AN

## How to apply

Load the supplementary Excel tables S8–S13 (CKD plasma cohort annotation results for CP, CN, AP, AN) from the academicCQMD/LongitudinalProf-MSMS-Method repository using pandas in a Python 3.8.13 Jupyter environment. For each mode, rank candidate metabolites retrieved from KEGG/HMDB with two matchers: the optimized Jaccard-similarity longitudinal fragment profiler at its ROC-derived 0.5 threshold, and the cross-sectional optimal-entropy matcher at its ROC-derived 0.75 threshold. Count correct top-ranked annotations per mode, confirming the reported totals of 518 (longitudinal) versus 485 (entropy) correct annotations, then compute per-mode accuracy and false discovery rate. Expect accuracies of 77.4/85.5/83.3/83.2% (CP/CN/AP/AN) for longitudinal profiling versus 61.5/75.9/76.6/71.2% for entropy matching, i.e. improvements of 25.9/12.6/8.7/16.9% and FDR reductions of 41.3/39.8/28.6/41.7%. Write per-mode results to figure5cd_reproduction.csv and verify they match the reported Figure 5c,d values.

## Related tools

- **Jaccard similarity algorithm** (Optimized longitudinal fragment-profile matcher applied at the 0.5 ROC-derived threshold to rank candidate metabolites per mode)
- **001-007.ipynb** (Jupyter notebook series producing the cohort annotation tables and running isomer similarity calculations through metabolite identification) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **mzbatch.py** (Mass-spectrometry data processing library used for spectrum extraction, filtering, and data calculation steps feeding the cohort comparison) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **Python 3.8.13** (Runtime environment (conda env 'longprof') in which the reproduction and per-mode metric computation are executed) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb
```

## Evaluation signals

- Correct top-ranked annotation totals must equal 518 for longitudinal profiling and 485 for cross-sectional entropy matching across the cohort
- Per-mode accuracies must reproduce 77.4/85.5/83.3/83.2% (longitudinal, 0.5 threshold) and 61.5/75.9/76.6/71.2% (entropy, 0.75 threshold) for CP/CN/AP/AN
- Derived accuracy improvements must match 25.9/12.6/8.7/16.9% and FDR reductions 41.3/39.8/28.6/41.7% across the four modes
- figure5cd_reproduction.csv values must match the reported Figure 5c,d values
- Thresholds must be the ROC-derived values (0.5 longitudinal, 0.75 entropy), not arbitrary cutoffs

## Limitations

- The in-house CQMD database used for ROC threshold derivation is not publicly available (no accession given), so thresholds cannot be re-derived independently
- Reported results reference supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) whose availability/location is not confirmed in all sections
- The repository README states the data are for experimental reference only and should not be used to guide human medicine
- No changelog is available for the repository, so code-version drift is hard to track

## Evidence

- [results] longitudinal profiling at 0.5 threshold achieved accuracies of 77.4, 85.5, 83.3, and 83.2% (CP, CN, AP, AN) versus 61.5, 75.9, 76.6, and 71.2% for cross-sectional entropy at 0.75 threshold: [evidence span withheld — anchor status: not_found]
- [abstract] improved annotation accuracy by 8.7-25.9% and reduced the false discovery rate by 28.6-41.7% compared with cross-sectional profiling: [evidence span withheld — anchor status: not_found]
- [methods] Compound candidates were retrieved from the KEGG and HMDB databases.: [evidence span withheld — anchor status: not_found]
