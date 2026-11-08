---
name: msms-fragment-profile-matching
description: Use when untargeted LC-MS/MS data are acquired at multiple collision
  energies (e.g., stepped NCE 20/40/60%) and isomers with similar substructures must
  be distinguished, so a single cross-sectional spectrum match is insufficient.
when_to_use_negative:
- When MS/MS data were acquired at only a single collision energy — no longitudinal
  fragment profiles exist, so cross-sectional matching (dot product or optimal entropy)
  is the appropriate method
- When raw MS/MS spectra have not been preprocessed (spectrum filtering, removal of
  invalid fragments, peaks with relative intensity below 1%, fragment rationality
  assessment) — LFPs computed on unfiltered spectra are unreliable
- When the goal is library-independent de novo annotation without reference longitudinal
  profiles (e.g., no CQMU/in-house library entries for the feature's precursor within
  10 ppm)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm (optimized)
  role: Computes similarity between experimental and reference longitudinal fragment
    profiles for each fragment across collision energies
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: mzbatch.py
  role: Python library of mass spectrometry data-processing methods (file operations,
    data validation, calculation, extraction) used to implement the matching workflow
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: 001-007.ipynb
  role: Notebook series covering spectrum extraction through isomer similarity calculations
    and metabolite identification
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: Python 3.8.13 / conda environment 'longprof'
  role: Runtime environment in which the notebooks and mzbatch.py functions are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:1e6aefe0f8000d37883f2f37ee27ede4a6ecf1d6d3a187b6f289c9ca121cd90c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm (optimized)
  - Python
  - Python 3.8.13 / conda environment 'longprof'
  - mzbatch.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# msms-fragment-profile-matching

## Summary

Identifies metabolites by matching longitudinal fragment profiles (LFPs) — relative ion-intensity traces of each fragment across multiple collision energies — between experimental and reference MS/MS spectra using an optimized Jaccard similarity and a criterion-factor ranking, which improves accuracy and lowers false discovery versus cross-sectional spectral matching.

## When to use

Use when untargeted LC-MS/MS data are acquired at multiple collision energies (e.g., stepped NCE 20/40/60%) and isomers with similar substructures must be distinguished, so a single cross-sectional spectrum match is insufficient.

## When NOT to use

- When MS/MS data were acquired at only a single collision energy — no longitudinal fragment profiles exist, so cross-sectional matching (dot product or optimal entropy) is the appropriate method
- When raw MS/MS spectra have not been preprocessed (spectrum filtering, removal of invalid fragments, peaks with relative intensity below 1%, fragment rationality assessment) — LFPs computed on unfiltered spectra are unreliable
- When the goal is library-independent de novo annotation without reference longitudinal profiles (e.g., no CQMU/in-house library entries for the feature's precursor within 10 ppm)

## Inputs

- Detected feature table with precursor m/z per metabolic feature (e.g., from Progenesis QI / MS-DIAL / Compound Discoverer exports)
- Experimental multi-collision-energy MS/MS spectra (MS1 and MS/MS spectrum extraction from 001-007.ipynb preprocessing)
- Reference longitudinal fragment profiles from a spectral database (in-house ~180,000 MS/MS spectra; CQMU database entries; KEGG/HMDB candidate compounds)

## Outputs

- candidate_rankings.csv — ranked candidate matches per feature, ordered per feature and per criterion factor
- Per-feature metabolite identifications with criterion-factor values in the 0.1–0.95 evaluated range

## How to apply

Load the detected feature table (precursor m/z per feature) and reference candidate entries (e.g., CQMD/CQMU or KEGG/HMDB-derived) with pandas/numpy, retaining only candidates whose precursor mass matches each feature within a 10 ppm tolerance. For each retained candidate, compute the Jaccard similarity between the experimental and reference longitudinal fragment profiles, where each fragment's profile is its relative ion intensity across the multiple collision energies. Count the required LFPs whose Jaccard score exceeds 0.5 and compute the criterion factor as the ratio of satisfied required LFPs to the total required LFPs; the candidate with the highest criterion factor is taken as the match. Evaluate candidates at eight criterion-factor thresholds spanning 0.1 to 0.95, applying the tie-breaking hierarchy: prefer the candidate satisfying more qualifying LFPs, then the forefront-ranking rule, then the larger summed score. Identification criteria should be established by ROC analysis (as done on thousands of isomers in the in-house CQMD database) before applying the thresholds to real samples such as human plasma.

## Related tools

- **Jaccard similarity algorithm (optimized)** (Computes similarity between experimental and reference longitudinal fragment profiles for each fragment across collision energies) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **mzbatch.py** (Python library of mass spectrometry data-processing methods (file operations, data validation, calculation, extraction) used to implement the matching workflow) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **001-007.ipynb** (Notebook series covering spectrum extraction through isomer similarity calculations and metabolite identification) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **Python 3.8.13 / conda environment 'longprof'** (Runtime environment in which the notebooks and mzbatch.py functions are executed)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt; jupyter notebook 007.ipynb  # run 001-007.ipynb in sequence to reach isomer similarity calculations and metabolite identification
```

## Evaluation signals

- All criterion-factor values in candidate_rankings.csv fall within the evaluated 0.1–0.95 threshold range
- Every retained candidate passed precursor-mass matching within the 10 ppm tolerance
- LFP matches require a Jaccard similarity score above 0.5; qualifying-LFP counts and the resulting criterion factor must be internally consistent (numerator ≤ denominator)
- Tie-breaking follows the documented hierarchy (more qualifying LFPs, then forefront-ranking rule, then larger summed score)
- Output file is parseable and contains a ranking column ordered per feature and per criterion factor

## Limitations

- The reference in-house database CQMD (~180,000 MS/MS spectra of >4500 metabolites) is described as in-house with no public accession, limiting external reuse
- Peaks with relative intensity below 1% are removed and common neutral-loss fragments are filtered before scoring, so very-low-abundance structural information is discarded
- The project's data and findings are intended for experimental reference only and should not be directly interpreted or used to guide human medicine (per README disclaimer)

## Evidence

- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
