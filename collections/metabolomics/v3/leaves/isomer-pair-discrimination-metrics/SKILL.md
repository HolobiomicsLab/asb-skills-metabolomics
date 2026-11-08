---
name: isomer-pair-discrimination-metrics
description: Use when you need to quantify sensitivity/specificity of spectral similarity
  scores for discriminating known isomer pairs (e.g., CQMD) across individual NCEs
  or stepped NCE conditions.
when_to_use_negative:
- Goal is to evaluate the longitudinal fragment-profile (LFP) matching method itself
  — this skill computes the cross-sectional baseline it is benchmarked against, not
  the LFP scores
- Input spectra were acquired at only a single collision energy with no multi-NCE
  or stepped NCE coverage, so the algorithm x NCE comparison cannot be constructed
- The metabolite pairs are not known isomers/non-isomers, so sensitivity/specificity
  against ground truth cannot be defined
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzbatch.py
  role: Library of mass-spectrometry data-processing methods used to compute similarity
    scores on isomer-pair spectra
- name: 001-007.ipynb
  role: Notebook series containing the isomer similarity calculations step of the
    workflow
- name: Jaccard similarity algorithm
  role: One of the four similarity algorithms scored; the paper's optimized variant
    for longitudinal profiling is compared here in cross-sectional mode
- name: LongitudinalProf-MSMS-Method
  role: Repository providing the notebooks, mzbatch.py, and requirements.txt needed
    to reproduce the metrics
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:18018a0ad8359b6a25a65affec618604eefc4ebe158f1b4cc4f0d4a7beb5cda7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - LongitudinalProf-MSMS-Method
  - Python
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 21
---

# isomer-pair-discrimination-metrics

## Summary

Benchmark how well MS/MS similarity algorithms (dot product, modified dot product, entropy, optimized Jaccard) separate isomer vs non-isomer pairs by computing sensitivity and specificity at a 0.95 score threshold across collision-energy conditions, providing the quantitative baseline that longitudinal profiling is compared against.

## When to use

Use when you need to quantify sensitivity/specificity of spectral similarity scores for discriminating known isomer pairs (e.g., CQMD) across individual NCEs or stepped NCE conditions.

## When NOT to use

- Goal is to evaluate the longitudinal fragment-profile (LFP) matching method itself — this skill computes the cross-sectional baseline it is benchmarked against, not the LFP scores
- Input spectra were acquired at only a single collision energy with no multi-NCE or stepped NCE coverage, so the algorithm x NCE comparison cannot be constructed
- The metabolite pairs are not known isomers/non-isomers, so sensitivity/specificity against ground truth cannot be defined

## Inputs

- CQMD isomer-pair MS/MS spectra processed through CQMU database-construction steps
- List of isomer/non-isomer pair labels for confusion-matrix computation
- Simulated/processed spectra per NCE condition (13 individual NCEs and stepped NCE 20/40/60%)

## Outputs

- cross_sectional_similarity_metrics.csv: full algorithm x NCE table of sensitivity and specificity
- Reference-condition metrics: NCE 70% (93.25%/87.96%), stepped NCE (92.25%/77.38%), entropy CP-mode (100%/50.83%), dot product CP-mode (99.80%/83.77%)

## How to apply

Load isomer-pair MS/MS spectra (CQMD pairs processed through the upstream CQMU database-construction steps) and compute cross-sectional similarity scores with four algorithms — dot product, modified dot product, entropy, and the optimized Jaccard similarity algorithm — at each of 13 individual NCEs and stepped NCE 20/40/60%. Binarize scores at the 0.95 threshold and compute sensitivity and specificity per algorithm x NCE condition over isomer/non-isomer pairs. Extract and validate the reference conditions: NCE 70% (sensitivity 93.25%, specificity 87.96%), stepped NCE 20/40/60% (92.25%/77.38%), entropy CP-mode (100%/50.83%), and dot product CP-mode (99.80%/83.77%). Write the full algorithm x NCE metrics table to CSV for downstream comparison with the longitudinal profiling results (98.89%/92.66%).

## Related tools

- **mzbatch.py** (Library of mass-spectrometry data-processing methods used to compute similarity scores on isomer-pair spectra)
- **001-007.ipynb** (Notebook series containing the isomer similarity calculations step of the workflow)
- **Jaccard similarity algorithm** (One of the four similarity algorithms scored; the paper's optimized variant for longitudinal profiling is compared here in cross-sectional mode)
- **LongitudinalProf-MSMS-Method** (Repository providing the notebooks, mzbatch.py, and requirements.txt needed to reproduce the metrics) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 006-isomer_similarity.ipynb
```

## Evaluation signals

- Recomputed NCE 70% metrics match sensitivity 93.25% and specificity 87.96% within numeric tolerance
- Recomputed stepped NCE 20/40/60% metrics match 92.25% sensitivity and 77.38% specificity
- Entropy CP-mode reproduces 100%/50.83% and dot product CP-mode reproduces 99.80%/83.77%
- Binarization threshold is exactly 0.95 for all algorithm x NCE conditions
- Output CSV contains one sensitivity/specificity row per algorithm x NCE combination across the four acquisition modes

## Limitations

- Cross-sectional evaluation performs worse than longitudinal profiling (98.89%/92.66%): cross-sectional scoring weighs predominant fragments heavily and diminishes minute fragments that reflect subtle structural differences, causing misidentifications for isomers with highly similar substructures
- The in-house CQMD isomer database is described as in-house with no public accession, limiting external validation
- Stepped NCE 20/40/60% is inferior to the optimal single NCE 70% for cross-sectional discrimination

## Evidence

- [results] Cross-sectional evaluation of CQMD isomer pairs performed best at NCE 70% with a 0.95 threshold, achieving an average sensitivity of 93.25% and a specificity of 87.96% across four modes: [evidence span withheld — anchor status: not_found]
