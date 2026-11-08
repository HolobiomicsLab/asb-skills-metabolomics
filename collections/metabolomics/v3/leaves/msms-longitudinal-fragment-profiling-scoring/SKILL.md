---
name: msms-longitudinal-fragment-profiling-scoring
description: Use when annotating metabolites from MS/MS spectra acquired at multiple
  distinct collision energies (e.g., stepped NCE 20/40/60%) and you need per-candidate
  similarity ranking plus ROC-derived identification thresholds.
when_to_use_negative:
- Only a single collision energy per compound is available — no longitudinal profile
  across multiple CEs can be constructed, so cross-sectional scoring (dot product
  or optimal entropy) is required instead
- Input is a feature table without MS/MS spectra or fragment intensity data — LFP
  matching needs per-fragment relative ion intensities at each CE
- No reference spectral library or candidate database (e.g., KEGG, HMDB, MoNA, GNPS,
  METLIN) is available for the metabolites of interest
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm (optimized)
  role: Similarity metric scoring longitudinal fragment profiles against reference
    spectra; ROC-derived 0.5 threshold applied for identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series executing spectrum extraction, filtering, invalid fragment
    removal, fragment rationality assessment, CQMU database construction, isomer similarity
    calculations, and metabolite identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of mass spectrometry data processing methods (file operations, data
    validation, generation, calculation, extraction) supporting the pipeline
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jupyter Notebook
  role: Environment in which 001-007.ipynb cells are run in sequence to reproduce
    the results
- name: Python 3.8.13
  role: Runtime for the pipeline, provisioned via conda environment 'longprof'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:49d90f585f463323538cfe567d79a03aff5e55abbda17b20fdf15690663e2db7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm (optimized)
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.84
n_claims: 25
---

# msms-longitudinal-fragment-profiling-scoring

## Summary

Scores candidate metabolites by matching longitudinal fragment profiles (LFPs — relative ion intensities of a fragment across multiple collision energies) with an optimized Jaccard similarity algorithm, instead of cross-sectional single-spectrum matching, to improve annotation accuracy and reduce false discovery rate in untargeted metabolomics.

## When to use

Use when annotating metabolites from MS/MS spectra acquired at multiple distinct collision energies (e.g., stepped NCE 20/40/60%) and you need per-candidate similarity ranking plus ROC-derived identification thresholds.

## When NOT to use

- Only a single collision energy per compound is available — no longitudinal profile across multiple CEs can be constructed, so cross-sectional scoring (dot product or optimal entropy) is required instead
- Input is a feature table without MS/MS spectra or fragment intensity data — LFP matching needs per-fragment relative ion intensities at each CE
- No reference spectral library or candidate database (e.g., KEGG, HMDB, MoNA, GNPS, METLIN) is available for the metabolites of interest

## Inputs

- MS/MS spectra acquired at 13 distinct collision energies (stepped NCE 20/40/60%)
- In-house spectral library (~180,000 MS/MS spectra of >4500 metabolites, CQMD/CQMU databases)
- Candidate compound lists from KEGG and HMDB
- Supplementary Excel tables S8–S13 (CKD plasma cohort annotation results for CP, CN, AP, AN modes)
- Repository files 001-007.ipynb, mzbatch.py, requirements.txt from LongitudinalProf-MSMS-Method

## Outputs

- Ranked candidate metabolites per acquisition mode (CP, CN, AP, AN) with Jaccard longitudinal-profile similarity scores
- Correct top-ranked annotation counts (expected: 518 for longitudinal profiling vs 485 for cross-sectional entropy on the CKD plasma cohort)
- Per-mode accuracy and FDR values (e.g., written to figure5cd_reproduction.csv)

## How to apply

Build longitudinal fragment profiles for each candidate from MS/MS spectra acquired at multiple CEs, weighing predominant and minute fragments independently since minute fragments carry the subtle structural differences that cross-sectional scoring (dot product or optimal entropy) diminishes. Rank candidates per mode (CP, CN, AP, AN) with the optimized Jaccard similarity matcher at its ROC-derived 0.5 threshold, derived from ROC analysis of thousands of isomers in the in-house CQMD database (~180,000 MS/MS spectra of >4500 metabolites). Preprocess with the repository's 001-007.ipynb series: MS1/MS/MS spectrum extraction, spectrum filtering, invalid fragment removal, fragment rationality assessment, CQMU database construction, then isomer similarity calculations. Compare against a cross-sectional entropy matcher at its recommended 0.75 threshold and compute per-mode accuracy and FDR to verify the expected gains (accuracy +8.7–25.9%, FDR −28.6–41.7% on human plasma).

## Related tools

- **Jaccard similarity algorithm (optimized)** (Similarity metric scoring longitudinal fragment profiles against reference spectra; ROC-derived 0.5 threshold applied for identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series executing spectrum extraction, filtering, invalid fragment removal, fragment rationality assessment, CQMU database construction, isomer similarity calculations, and metabolite identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of mass spectrometry data processing methods (file operations, data validation, generation, calculation, extraction) supporting the pipeline) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jupyter Notebook** (Environment in which 001-007.ipynb cells are run in sequence to reproduce the results)
- **Python 3.8.13** (Runtime for the pipeline, provisioned via conda environment 'longprof')

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt
```

## Evaluation signals

- Correct top-ranked annotation counts on the CKD plasma cohort equal 518 for longitudinal profiling and 485 for cross-sectional entropy matching
- Reproduced per-mode accuracies match Figure 5c,d values: 77.4/85.5/83.3/83.2% (longitudinal, 0.5 threshold) vs 61.5/75.9/76.6/71.2% (entropy, 0.75 threshold) for CP/CN/AP/AN
- Accuracy improvements equal 25.9/12.6/8.7/16.9% and FDR reductions equal 41.3/39.8/28.6/41.7% across CP/CN/AP/AN
- Thresholds used are the ROC-derived values: 0.5 for the Jaccard longitudinal matcher and 0.75 for the optimal-entropy cross-sectional matcher
- figure5cd_reproduction.csv is written and its rows match the reported per-mode accuracy, improvement, and FDR values

## Limitations

- The in-house CQMD database availability is not stated — described as in-house with no accession or public access given
- Data are intended for experimental reference only and should not be directly interpreted or used to guide human medicine (repository license disclaimer)
- Accuracy gains are largest for CP mode (25.9%) and smallest for AP mode (8.7%); isomers with highly similar substructures remain the hard cases that motivated the method

## Evidence

- [methods] methods: [evidence span withheld — anchor status: not_found]
- [discussion] discussion: [evidence span withheld — anchor status: not_found]
