---
name: jaccard-similarity-scoring-of-longitudinal-fragment-profiles
description: Use when annotating metabolites from MS/MS spectra acquired at multiple
  distinct collision energies (e.g. 13 CEs or stepped NCE 20/40/60%), where cross-sectional
  scoring misranks isomers with similar predominant fragments.
when_to_use_negative:
- When MS/MS spectra are only available at a single collision energy — no longitudinal
  profiles can be constructed and cross-sectional matching (dot product or optimal
  entropy) should be used instead
- When the goal is feature detection or peak picking rather than spectral matching
  of already-extracted features against a candidate library
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: Core scoring algorithm optimized for longitudinal profiling of MS/MS fragments
- name: mzbatch.py
  role: Python library of mass spectrometry data processing methods used for file
    operations, data validation, data generation, data calculation, and data extraction
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series implementing MS1 and MS/MS spectrum extraction, spectrum filtering,
    invalid fragment removal, fragment rationality assessment, CQMU database construction,
    isomer similarity calculations, and metabolite identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') for executing the pipeline
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:279d4d0eb26fd0380a31d3304674317e87b2c2c94f5b6264074a229f754384e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# Jaccard similarity scoring of longitudinal fragment profiles

## Summary

An optimized Jaccard similarity algorithm that scores metabolite candidates by comparing experimental versus reference longitudinal fragment profiles (LFPs) — relative ion intensities of each fragment across multiple collision energies — treating predominant and minute fragments independently rather than pooling them as in cross-sectional spectral matching.

## When to use

Use when annotating metabolites from MS/MS spectra acquired at multiple distinct collision energies (e.g. 13 CEs or stepped NCE 20/40/60%), where cross-sectional scoring misranks isomers with similar predominant fragments.

## When NOT to use

- When MS/MS spectra are only available at a single collision energy — no longitudinal profiles can be constructed and cross-sectional matching (dot product or optimal entropy) should be used instead
- When the goal is feature detection or peak picking rather than spectral matching of already-extracted features against a candidate library

## Inputs

- Experimental MS/MS spectra acquired at multiple distinct collision energies (e.g. 13 CEs or stepped NCE 20/40/60%)
- Precursor m/z feature table of detected metabolic features
- Candidate compound entries from a reference database (KEGG, HMDB, CQMD/CQMU)
- Reference spectral library of LFPs (e.g. in-house database of ~180,000 MS/MS spectra of over 4500 metabolites)

## Outputs

- Ranked candidate list per feature with Jaccard similarity scores and criterion factors
- Metabolite annotations with false discovery rate estimates validated by ROC analysis

## How to apply

Extract precursor and MS/MS spectra, filter peaks with relative intensity below 1%, remove invalid fragments, and build longitudinal fragment profiles capturing each fragment's relative ion intensity across the multiple collision energies. For each detected feature, retain candidate compounds (e.g. from KEGG/HMDB or CQMD) whose precursor mass matches within 10 ppm, then compute the optimized Jaccard similarity between experimental and reference LFPs. Rank candidates by the criterion factor — the ratio of LFPs with a Jaccard score above 0.5 to the required number of such LFPs — evaluating thresholds from 0.1 to 0.95 with tie-breaking by more qualifying LFPs, then forefront ranking, then larger summed score. Identification criteria should be established by ROC analysis against thousands of isomers in the in-house CQMD database. The rationale is that cross-sectional scoring weighs predominant fragments heavily and diminishes minute fragments that reflect subtle structural differences; longitudinal scoring considers them independently, improving annotation accuracy.

## Related tools

- **Jaccard similarity algorithm** (Core scoring algorithm optimized for longitudinal profiling of MS/MS fragments)
- **mzbatch.py** (Python library of mass spectrometry data processing methods used for file operations, data validation, data generation, data calculation, and data extraction) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series implementing MS1 and MS/MS spectrum extraction, spectrum filtering, invalid fragment removal, fragment rationality assessment, CQMU database construction, isomer similarity calculations, and metabolite identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime environment (conda env 'longprof') for executing the pipeline) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && jupyter notebook 001-007.ipynb
```

## Evaluation signals

- Jaccard similarity scores for every compared LFP fall in [0, 1], and the evaluated criterion-factor thresholds span 0.1–0.95
- Retained candidates satisfy precursor mass matching within 10 ppm of the detected feature's m/z
- Annotation accuracy on validation data improves over cross-sectional profiling with the optimal entropy algorithm (expected gain 8.7–25.9% in human plasma across four modes)
- False discovery rate decreases by 28.6–41.7% relative to cross-sectional matching
- ROC analysis of thousands of CQMD isomers shows better sensitivity and specificity than existing cross-sectional matching methods

## Limitations

- Requires MS/MS acquisition at multiple distinct collision energies; spectra from a single CE cannot form longitudinal profiles
- The in-house CQMD database is described as in-house with no accession or public access stated, limiting full reproducibility of the ROC-derived criteria
- Peaks with relative intensity below 1% are removed, so very low-abundance fragments cannot contribute to the LFPs

## Evidence

- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
