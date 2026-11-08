---
name: metabolite-candidate-ranking-with-tie-breaking
description: Use when assigning a compound identity to each detected precursor-m/z
  feature by matching its experimental longitudinal fragment profiles (multi-collision-energy
  MS/MS) against CQMD candidates and ranking them by criterion factor.
when_to_use_negative:
- Only a single cross-sectional MS/MS spectrum per feature is available (no multi-collision-energy
  longitudinal profiles), since LFP Jaccard similarity across CEs cannot be computed.
- Candidates cannot be precursor-mass-matched to the feature within the 10 ppm tolerance
  — they are excluded before ranking and there is nothing to score.
- The goal is de novo structure elucidation rather than candidate identification/ranking
  against a reference database.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: Optimized similarity metric for scoring experimental vs reference longitudinal
    fragment profiles per candidate
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') in which the candidate filtering,
    Jaccard scoring, and criterion-factor ranking are implemented
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of mass-spectrometry data-processing methods (data validation, calculation,
    extraction) used in the ranking workflow
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series covering spectrum extraction through isomer similarity calculations
    to metabolite identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:e45080a60d879d006fdd11a10b17892a943e09672a7a59696a9750ae8c1871aa
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metabolite-candidate-ranking-with-tie-breaking

## Summary

Ranks CQMD candidate metabolites against each detected metabolic feature by a criterion factor — the fraction of required longitudinal fragment profiles (LFPs) whose Jaccard similarity to the experimental reference exceeds 0.5 — and resolves ties via a fixed hierarchy rather than raw score alone.

## When to use

Use when assigning a compound identity to each detected precursor-m/z feature by matching its experimental longitudinal fragment profiles (multi-collision-energy MS/MS) against CQMD candidates and ranking them by criterion factor.

## When NOT to use

- Only a single cross-sectional MS/MS spectrum per feature is available (no multi-collision-energy longitudinal profiles), since LFP Jaccard similarity across CEs cannot be computed.
- Candidates cannot be precursor-mass-matched to the feature within the 10 ppm tolerance — they are excluded before ranking and there is nothing to score.
- The goal is de novo structure elucidation rather than candidate identification/ranking against a reference database.

## Inputs

- Detected feature table with precursor m/z per metabolic feature
- CQMD candidate entries (compound candidates, e.g. from KEGG/HMDB-backed in-house database)
- Experimental longitudinal fragment profiles (fragment relative ion intensities across multiple collision energies, 13 distinct CEs / stepped NCE 20/40/60% in the paper)
- Reference LFPs from the CQMU database (constructed by 001-007.ipynb notebooks)

## Outputs

- candidate_rankings.csv — ranked candidate results ordered per feature and per criterion factor (thresholds 0.1–0.95)

## How to apply

Load the detected feature table (precursor m/z per feature) and CQMD candidate entries in Python (pandas/numpy). For each feature, retain only candidates whose precursor mass matches within a 10 ppm tolerance, then compute the Jaccard similarity between the experimental and reference longitudinal fragment profiles — the relative ion-intensity profiles of each fragment across multiple collision energies. Count the required LFPs whose Jaccard score exceeds 0.5 and compute the criterion factor as the ratio of qualifying LFPs to the number of LFPs required by the criteria. Evaluate candidates at eight criterion-factor thresholds spanning 0.1 to 0.95, applying the tie-breaking hierarchy: prefer the candidate satisfying more qualifying LFPs, then the forefront-ranking rule, then the larger summed score; the candidate with the highest criterion factor is taken as the match. Write the ranked results, ordered per feature and per criterion factor, to candidate_rankings.csv.

## Related tools

- **Jaccard similarity algorithm** (Optimized similarity metric for scoring experimental vs reference longitudinal fragment profiles per candidate) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime environment (conda env 'longprof') in which the candidate filtering, Jaccard scoring, and criterion-factor ranking are implemented) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of mass-spectrometry data-processing methods (data validation, calculation, extraction) used in the ranking workflow) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series covering spectrum extraction through isomer similarity calculations to metabolite identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda activate longprof && jupyter nbconvert --to notebook --execute 007.ipynb  # runs candidate matching/ranking, writing candidate_rankings.csv
```

## Evaluation signals

- candidate_rankings.csv exists and parses as a table containing a ranking column.
- All criterion-factor values in the output fall within the evaluated 0.1–0.95 threshold range.
- Every candidate scored for a feature was retained only after precursor-mass matching within the 10 ppm tolerance.
- All Jaccard similarity-based criterion factors are ratios of qualifying LFPs (score > 0.5) to required LFPs, so factors lie in [0, 1].
- Tie-breaking consistency reviewed by expert inspection: more qualifying LFPs preferred, then forefront-ranking rule, then larger summed score.

## Limitations

- The in-house CQMD database used for ROC-derived criteria is not publicly available (no accession or access path given).
- The similarity threshold of 0.5 and the eight criterion-factor cutoffs (0.1–0.95) are ROC-derived on the CQMD isomer set; they may not transfer directly to other instruments or libraries.
- Candidates tied in LFP satisfaction require the forefront-ranking and summed-score rules, whose implementation details live only in the supplementary notebooks.
- Identifications remain database-dependent; the method matches against reference spectra rather than confirming structure with authentic standards.

## Evidence

- [results] Candidates are ranked by a criterion factor, defined as the ratio of the number of LFPs with a similarity score over 0.5 within the criteria to the required number of LFPs: [evidence span withheld — anchor status: not_found]
- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
