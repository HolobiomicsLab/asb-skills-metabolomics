---
name: criterion-factor-threshold-evaluation
description: Use when a detected feature (precursor m/z) has multiple CQMD/candidate
  metabolites and you must rank them by how many required longitudinal fragment profiles
  meet the Jaccard > 0.5 similarity criteria.
when_to_use_negative:
- MS/MS spectra were acquired at only a single collision energy, so no longitudinal
  (multi-CE) fragment profiles exist to compute Jaccard similarity against
- The goal is cross-sectional spectral matching with dot-product or optimal-entropy
  scoring rather than ranking candidates by LFP criterion factors
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: Optimized similarity metric used to score experimental vs. reference longitudinal
    fragment profiles (LFPs) at the >0.5 cutoff
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') in which the pandas/numpy candidate
    filtering, Jaccard scoring, and criterion-factor ranking are executed
- name: mzbatch.py
  role: Library of mass spectrometry data-processing methods (file operations, data
    validation/calculation/extraction) supporting the ranking workflow
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:3e9e77dddb78c12d03eb7eb2e756aa02792c5bab3e80b7272a15c7784a5bb988
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - Jaccard similarity algorithm
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# criterion-factor-threshold-evaluation

## Summary

Ranks metabolite candidates against each detected metabolic feature by a criterion factor — the fraction of required longitudinal fragment profiles (LFPs) whose Jaccard similarity exceeds 0.5 — evaluated at eight thresholds spanning 0.1–0.95, as defined in the longitudinal profiling method (Anal. Chem. 2025, DOI 10.1021/acs.analchem.5c01414).

## When to use

Use when a detected feature (precursor m/z) has multiple CQMD/candidate metabolites and you must rank them by how many required longitudinal fragment profiles meet the Jaccard > 0.5 similarity criteria.

## When NOT to use

- MS/MS spectra were acquired at only a single collision energy, so no longitudinal (multi-CE) fragment profiles exist to compute Jaccard similarity against
- The goal is cross-sectional spectral matching with dot-product or optimal-entropy scoring rather than ranking candidates by LFP criterion factors

## Inputs

- Detected feature table (precursor m/z per metabolic feature)
- CQMD candidate metabolite entries with reference longitudinal fragment profiles (relative ion-intensity profiles across multiple collision energies)

## Outputs

- candidate_rankings.csv — ranked candidate results, ordered per feature and per criterion factor

## How to apply

Load the detected feature table (precursor m/z per feature) and CQMD candidate entries with pandas/numpy, retaining only candidates whose precursor mass matches the feature within a 10 ppm mass tolerance. For each retained candidate, compute the Jaccard similarity between the experimental and reference longitudinal fragment profiles — the relative ion-intensity profiles of each fragment across multiple collision energies — then count required LFPs with a Jaccard score above 0.5 and express the criterion factor as the fraction of required LFPs satisfied. Evaluate candidates at eight criterion-factor thresholds spanning 0.1 to 0.95, taking the candidate with the highest criterion factor as the match; break ties by preferring the candidate satisfying more qualifying LFPs, then the forefront-ranking rule, then the larger summed score. Write ranked results, ordered per feature and per criterion factor, to candidate_rankings.csv.

## Related tools

- **Jaccard similarity algorithm** (Optimized similarity metric used to score experimental vs. reference longitudinal fragment profiles (LFPs) at the >0.5 cutoff)
- **Python 3.8.13** (Runtime environment (conda env 'longprof') in which the pandas/numpy candidate filtering, Jaccard scoring, and criterion-factor ranking are executed)
- **mzbatch.py** (Library of mass spectrometry data-processing methods (file operations, data validation/calculation/extraction) supporting the ranking workflow) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt; jupyter nbconvert --to notebook --execute 006.ipynb  # candidate ranking to candidate_rankings.csv
```

## Evaluation signals

- candidate_rankings.csv exists and parses as a table with a ranking column
- All criterion-factor values in the output fall within the evaluated 0.1–0.95 range
- Each feature retains only candidates whose precursor mass matches within 10 ppm before scoring
- Per-feature top-ranked candidate has the highest criterion factor; ties resolved by more qualifying LFPs, then forefront-ranking, then larger summed score
- Required LFPs are counted only where the Jaccard similarity score exceeds 0.5

## Limitations

- The in-house CQMD database used for criterion establishment (ROC analysis) is described as in-house with no public accession stated
- Results referenced in supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) have unconfirmed availability/location
- README states the dataset and findings are intended for experimental reference only and should not be directly interpreted or used to guide human medicine; project is provided 'as is' without warranty

## Evidence

- [results] Candidates are ranked by a criterion factor, defined as the ratio of the number of LFPs with a similarity score over 0.5 within the criteria to the required number of LFPs with a similarity score above 0.5 by the criteria: [evidence span withheld — anchor status: not_found]
- [methods] precursor mass matches the feature within a 10 ppm mass tolerance: [evidence span withheld — anchor status: not_found]
- [supplementary] Evaluate the candidates at the eight criterion-factor thresholds spanning 0.1 to 0.95, applying the tie-breaking hierarchy: [evidence span withheld — anchor status: not_found]
