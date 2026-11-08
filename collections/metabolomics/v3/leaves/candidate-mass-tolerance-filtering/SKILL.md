---
name: candidate-mass-tolerance-filtering
description: Use when a detected MS1 metabolic feature (precursor m/z) must be matched
  against a candidate compound database (e.g., CQMD, KEGG, HMDB) before MS/MS-based
  LFP scoring, and candidates need pruning to a ppm-level mass window prior to similarity
  ranking.
when_to_use_negative:
- The MS/MS data were acquired at a single collision energy only, so no longitudinal
  fragment profile across multiple CEs can be constructed for Jaccard scoring
- The goal is de novo structure elucidation with no candidate database entries to
  filter, rather than ranking known candidates against detected features
- The feature table already contains similarity scores or a final ranking column —
  re-running the mass filter would discard the existing ranked output
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm
  role: Computes the similarity between experimental and reference longitudinal fragment
    profiles used to score each mass-matched candidate
- name: Python 3.8.13
  role: Environment for implementing the 10 ppm precursor-mass filter, criterion-factor
    calculation, and threshold evaluation with pandas/numpy
- name: mzbatch.py
  role: Library of mass spectrometry data-processing methods (file operations, data
    validation, calculation, extraction) supporting the filtering and ranking code
- name: 001-007.ipynb
  role: Notebook series executing the pipeline from spectrum extraction through isomer
    similarity calculations to metabolite identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:82811f0f4083b2424a9dfa09f4894e91e60746ec5a20924598847a91e600da5d
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
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# candidate-mass-tolerance-filtering

## Summary

A candidate filtering and ranking step for metabolite identification with longitudinal fragment profiles (LFPs): retain only database candidates whose precursor m/z matches a detected feature within a 10 ppm mass tolerance, then rank the survivors by the criterion factor derived from Jaccard-similarity matches of longitudinal fragment profiles.

## When to use

Use when a detected MS1 metabolic feature (precursor m/z) must be matched against a candidate compound database (e.g., CQMD, KEGG, HMDB) before MS/MS-based LFP scoring, and candidates need pruning to a ppm-level mass window prior to similarity ranking.

## When NOT to use

- The MS/MS data were acquired at a single collision energy only, so no longitudinal fragment profile across multiple CEs can be constructed for Jaccard scoring
- The goal is de novo structure elucidation with no candidate database entries to filter, rather than ranking known candidates against detected features
- The feature table already contains similarity scores or a final ranking column — re-running the mass filter would discard the existing ranked output

## Inputs

- Detected feature table with precursor m/z per metabolic feature
- CQMD candidate compound entries (candidates retrieved from KEGG and HMDB databases)
- Experimental MS/MS spectra acquired at multiple collision energies (13 distinct CEs / stepped NCE 20/40/60%)
- Reference longitudinal fragment profiles from the in-house spectral database (~180,000 MS/MS spectra of >4500 metabolites)

## Outputs

- candidate_rankings.csv — ranked candidate results, ordered per feature and per criterion factor
- Per-candidate criterion factor values spanning the evaluated 0.1–0.95 range

## How to apply

Load the detected feature table (precursor m/z per feature) and the CQMD candidate entries in Python (pandas/numpy). For each feature, retain only candidates whose precursor mass matches the feature within a 10 ppm mass tolerance. For each retained candidate, compute the Jaccard similarity between the experimental and reference longitudinal fragment profiles (relative ion-intensity profiles of each fragment across multiple collision energies), count the required LFPs whose score exceeds 0.5, and compute the criterion factor as the fraction of required LFPs satisfied. Evaluate candidates at the eight criterion-factor thresholds spanning 0.1 to 0.95, applying the tie-breaking hierarchy: prefer the candidate satisfying more qualifying LFPs, then the forefront-ranking rule, then the larger summed score. The rationale is that cross-sectional matching misidentifies isomers with highly similar substructures, so the 10 ppm pre-filter restricts the ranking space while LFP-based scoring discriminates among mass-matching isomers.

## Related tools

- **Jaccard similarity algorithm** (Computes the similarity between experimental and reference longitudinal fragment profiles used to score each mass-matched candidate)
- **Python 3.8.13** (Environment for implementing the 10 ppm precursor-mass filter, criterion-factor calculation, and threshold evaluation with pandas/numpy)
- **mzbatch.py** (Library of mass spectrometry data-processing methods (file operations, data validation, calculation, extraction) supporting the filtering and ranking code)
- **001-007.ipynb** (Notebook series executing the pipeline from spectrum extraction through isomer similarity calculations to metabolite identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb
```

## Evaluation signals

- candidate_rankings.csv exists and parses as a table with a ranking column
- All criterion-factor values in the output fall within the evaluated 0.1–0.95 range
- Every retained candidate's precursor mass matches its feature within the 10 ppm mass tolerance (spot-check a few features)
- Per feature, the top-ranked candidate is the one with the highest criterion factor, and ties resolve by more qualifying LFPs, then forefront-ranking rule, then larger summed score (expert review of tie-breaking consistency)
- Number of candidates per feature is reduced relative to the unfiltered CQMD entry list (mass filter visibly pruned the space)

## Limitations

- The in-house CQMD database is not publicly available (no accession or public access given), so candidate lists cannot be fully reproduced externally
- The criterion factor depends on the Jaccard similarity threshold of 0.5 and on required LFPs defined by the ROC-derived identification criteria; different thresholds change the ranking
- Isomers with highly similar substructures can still yield comparable scores and require the full tie-breaking hierarchy
- Repository data are provided 'as is' with no warranty and are for experimental reference only

## Evidence

- [full_text] 10 ppm pre-filter: [evidence span withheld — anchor status: not_found]
- [full_text] criterion factor definition: [evidence span withheld — anchor status: not_found]
- [methods] candidate sources: [evidence span withheld — anchor status: not_found]
- [intro] isomer misidentification problem: "This practice enables most metabolites to be correctly identified with the highest efficiency, but a considerable number of misidentifications still occur, particularly for the isomers with highly"
- [readme] pipeline scope: "This series of files includes key data processing steps, ranging from MS1 and MS/MS spectrum extraction, spectrum filtering, invalid fragment removal, fragment rationality assessment, CQMU database"
