---
name: annotation-fdr-computation
description: Use when comparing two metabolite-annotation matching strategies (e.g.,
  longitudinal Jaccard-similarity profiling at 0.5 threshold vs cross-sectional optimal-entropy
  at 0.75 threshold) and you need per-mode accuracy and FDR numbers to quantify identification
  gains.
when_to_use_negative:
- Input is raw MS/MS spectra or unranked feature tables — annotation candidates must
  already be scored and ranked by a matching method before accuracy/FDR can be computed
- The study lacks ground-truth identities (e.g., no in-house spectral database or
  spiked standards), so correct top-ranked annotations cannot be counted
- Data were acquired at a single collision energy only — longitudinal profiling and
  its multi-CE thresholds do not apply
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') for loading Excel tables and computing
    accuracy/FDR
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jupyter Notebook
  role: Interactive environment in which the reproduction notebooks (001-007.ipynb
    series) are executed cell-by-cell
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of mass spectrometry data-processing methods used for data calculation
    and extraction supporting the annotation counts
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jaccard similarity algorithm
  role: Optimized longitudinal fragment profiling matcher whose top-ranked annotations
    are scored at the ROC-derived 0.5 threshold
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:656d6e6240288d27ea5ede88b7908824fb0fea02520cda19114a3794f7b6841a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# annotation-fdr-computation

## Summary

Computes per-mode annotation accuracy and false discovery rate for metabolite identification (CP, CN, AP, AN) by counting correct top-ranked annotations under a chosen matching method and threshold, enabling direct comparison of longitudinal profiling vs cross-sectional entropy matching as in the CKD plasma cohort analysis.

## When to use

Use when comparing two metabolite-annotation matching strategies (e.g., longitudinal Jaccard-similarity profiling at 0.5 threshold vs cross-sectional optimal-entropy at 0.75 threshold) and you need per-mode accuracy and FDR numbers to quantify identification gains.

## When NOT to use

- Input is raw MS/MS spectra or unranked feature tables — annotation candidates must already be scored and ranked by a matching method before accuracy/FDR can be computed
- The study lacks ground-truth identities (e.g., no in-house spectral database or spiked standards), so correct top-ranked annotations cannot be counted
- Data were acquired at a single collision energy only — longitudinal profiling and its multi-CE thresholds do not apply

## Inputs

- Supplementary Excel tables S8–S13 (CKD plasma cohort annotation results for CP, CN, AP, AN modes)
- Ranked candidate metabolite annotation lists per mode from the Jaccard-similarity longitudinal profiler (0.5 threshold)
- Ranked candidate metabolite annotation lists per mode from the cross-sectional optimal-entropy matcher (0.75 threshold)
- Ground-truth correct top-ranked annotation counts per mode

## Outputs

- Per-mode accuracy values for each method (CP, CN, AP, AN)
- Per-mode false discovery rate values for each method
- Accuracy improvement percentages (25.9/12.6/8.7/16.9%) and FDR reduction percentages (41.3/39.8/28.6/41.7%)
- figure5cd_reproduction.csv with reproduced accuracy, improvement, and FDR values

## How to apply

Load the supplementary Excel tables S8–S13 containing CKD plasma cohort annotation results for CP, CN, AP, and AN modes from the academicCQMD/LongitudinalProf-MSMS-Method repository using pandas in Python 3.8.13 within Jupyter Notebook. For each of the four modes, rank candidate metabolites with the optimized Jaccard-similarity longitudinal fragment profiling matcher at its ROC-derived 0.5 threshold and with the cross-sectional optimal-entropy matcher at its ROC-derived 0.75 threshold. Count correct top-ranked annotations per mode per method, verifying the reported totals of 518 correct annotations for longitudinal profiling versus 485 for cross-sectional entropy matching. Divide correct counts by the total annotations to obtain accuracy, and derive the false discovery rate per mode for both methods; then compute the accuracy improvements (25.9/12.6/8.7/16.9% across CP/CN/AP/AN) and FDR reductions (41.3/39.8/28.6/41.7%). Write the reproduced values to figure5cd_reproduction.csv and check that accuracies match the reported Figure 5c,d values (77.4/85.5/83.3/83.2% longitudinal vs 61.5/75.9/76.6/71.2% entropy).

## Related tools

- **Python 3.8.13** (Runtime environment (conda env 'longprof') for loading Excel tables and computing accuracy/FDR) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jupyter Notebook** (Interactive environment in which the reproduction notebooks (001-007.ipynb series) are executed cell-by-cell) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of mass spectrometry data-processing methods used for data calculation and extraction supporting the annotation counts) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jaccard similarity algorithm** (Optimized longitudinal fragment profiling matcher whose top-ranked annotations are scored at the ROC-derived 0.5 threshold) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt; jupyter nbconvert --to notebook --execute 001-007.ipynb
```

## Evaluation signals

- Longitudinal profiling reproduces 518 correct top-ranked annotations total versus 485 for cross-sectional entropy matching
- Per-mode accuracies match Figure 5c,d values: 77.4/85.5/83.3/83.2% (longitudinal, 0.5 threshold) vs 61.5/75.9/76.6/71.2% (entropy, 0.75 threshold) for CP/CN/AP/AN
- Accuracy improvements fall at 25.9/12.6/8.7/16.9% and FDR reductions at 41.3/39.8/28.6/41.7% across CP/CN/AP/AN
- Reproduced values written to figure5cd_reproduction.csv match the reported figures within rounding error
- Thresholds used are the ROC-derived values (0.5 for longitudinal, 0.75 for optimal-entropy), not defaults

## Limitations

- The in-house CQMD database used for ROC-derived thresholds is described as in-house with no accession or public access given
- Results reference supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) whose availability/location is not confirmed in the discussion section
- README states the dataset and findings are for experimental reference only and should not be directly interpreted or used to guide human medicine

## Evidence

- [abstract] Longitudinal profiling accuracy/FDR reproduction on CKD plasma: "improved annotation accuracy by 8.7-25.9% and reduced the false discovery rate by 28.6-41.7%"
- [abstract] Optimized Jaccard similarity for longitudinal profiling: "We optimized the Jaccard similarity algorithm for longitudinal profiling"
- [discussion] ROC-derived identification criteria from CQMD isomers: "The identification criteria for the longitudinal profiling method were established based on the ROC analysis of the thousands of isomers in the in-house database CQMD"
- [readme] Notebook execution workflow: "Run every cell of "001-007.ipynb" in sequence in Jupyter Notebook. Make sure that the data set is stored in the same path as specified in the script."
- [readme] Environment setup for reproduction: "conda create -n longprof python=3.8.13"
