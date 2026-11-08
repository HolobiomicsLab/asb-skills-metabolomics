---
name: spectral-region-multiplet-matching
description: Use when a metabolite candidate table from NMR ROI analysis must be filtered
  across multiple spectral regions-of-interest, e.g. HMDB SRRMs matched against ROIs
  within 0.5-4.5 ppm of biofluid spectra.
when_to_use_negative:
- Automated ROI identification is the goal — ROIAL-NMR was not designed for this ('Automated
  identification of ROIs is beyond the scope of the current version of the program')
- Absolute quantification of metabolite concentrations is the aim — this is not the
  primary purpose of the program
- Validating the accuracy of user-defined ROIs — the program was not built to test
  ROI accuracy
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0593
tools:
- name: ROIAL-NMR
  role: Implements the cross-ROI match ratio filter (≥ 50% of SRRMs within analyzed
    ROIs) and outputs match ratios and matched regions in the 'All Metabolites' window
  repo: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- name: pandas
  role: Loads and processes the per-ROI candidate table and HMDB peak/SRRM database
    for match-ratio computation
- name: Human Metabolome Database (HMDB)
  role: Reference platform supplying chemical shift values and metabolite-specific
    SRRMs used in the match-ratio calculation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/nbm.70131
    title: Automatic Identification of Potential Cellular Metabolites for Untargeted
      NMR Metabolomics
  source_content_hash: sha256:ec0edcb7e8b0712bffc0fb1ed697f23952c56468bde6812901ff8f402bcce90b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
  tools:
  - Human Metabolome Database (HMDB)
  - Python
  - ROIAL-NMR
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# spectral-region-multiplet-matching

## Summary

A match-ratio-based metabolite filtering step from ROIAL-NMR that scores each candidate metabolite by the fraction of its SRRMs (metabolite-specific signal ranges) whose chemical shift peaks fall inside user-defined proton NMR ROIs, retaining only metabolites with match ratio ≥ 0.5 to avoid over-identification in crowded spectra.

## When to use

Use when a metabolite candidate table from NMR ROI analysis must be filtered across multiple spectral regions-of-interest, e.g. HMDB SRRMs matched against ROIs within 0.5-4.5 ppm of biofluid spectra.

## When NOT to use

- Automated ROI identification is the goal — ROIAL-NMR was not designed for this ('Automated identification of ROIs is beyond the scope of the current version of the program')
- Absolute quantification of metabolite concentrations is the aim — this is not the primary purpose of the program
- Validating the accuracy of user-defined ROIs — the program was not built to test ROI accuracy

## Inputs

- Per-ROI candidate metabolite table (from prior ROI matching step)
- HMDB-derived database of metabolite chemical shift peaks and metabolite-specific SRRMs
- User-defined ROI boundaries (ppm ranges, e.g. 18 ROIs within 0.5-4.5 ppm for serum)

## Outputs

- Scored metabolite table including match ratios and matched regions, saved as a tabular file (e.g. via XlsxWriter/openpyxl) for the categorization step
- Filtered set of metabolites with match ratio ≥ 0.5 carried forward to significance categorization

## How to apply

Load the per-ROI candidate metabolite table and the HMDB-derived database of chemical shift peaks and SRRMs (e.g. with pandas in the ROIAL-NMR Python environment). Pre-filter each metabolite by excluding peaks/SRRMs outside the analyzed region (0.5-4.5 ppm for biofluids); if more than 50% of a metabolite's peaks lie beyond the analyzed region, exclude those peaks from the match-ratio calculation. Apply the peak-inclusion rule: a SRRM is counted as matched to a ROI if at least one of its chemical shift peaks (e.g. lactate peaks at 1.324 and 4.111 ppm) falls within the ROI boundaries. Compute the match ratio as (number of SRRMs matched to ROIs) / (total SRRMs for that metabolite) and keep only metabolites with match ratio ≥ 0.5 (50%) for downstream categorization into the six trend/significance categories. The rationale is that no existing NMR database collectively identifies metabolites spanning multiple ROIs, and the cross-ROI match ratio prevents over-identification; validated by the lactate worked example where SRRMs [1.30-1.33] and [4.08-4.13] ppm match ROIs [1.31-1.34] and [4.10-4.12] ppm giving a ratio of 2/2 = 1.0.

## Related tools

- **ROIAL-NMR** (Implements the cross-ROI match ratio filter (≥ 50% of SRRMs within analyzed ROIs) and outputs match ratios and matched regions in the 'All Metabolites' window) — https://github.com/Leo-Cheng-Lab/ROIAL-NMR.git
- **pandas** (Loads and processes the per-ROI candidate table and HMDB peak/SRRM database for match-ratio computation)
- **Human Metabolome Database (HMDB)** (Reference platform supplying chemical shift values and metabolite-specific SRRMs used in the match-ratio calculation)

## Examples

```
python main.py
```

## Evaluation signals

- Lactate worked example reproduces a match ratio of 2/2 = 1.0 (SRRMs [1.30-1.33] and [4.08-4.13] ppm vs ROIs [1.31-1.34] and [4.10-4.12] ppm)
- Every retained metabolite has match ratio ≥ 0.5; any metabolite with < 50% of SRRMs within analyzed ROIs is excluded
- Peaks falling outside the analyzed region (0.5-4.5 ppm for biofluids) are excluded from the ratio, and if > 50% of a metabolite's peaks lie beyond the region those peaks are excluded from the calculation
- Output table contains per-metabolite match ratios and matched regions, matching the 'All Metabolites' window fields (abbreviations, match ratios, matched regions, concentration ranges)
- Runtime sanity check: single-population analysis of ~891 HMDB metabolites completes in about 5 seconds

## Limitations

- Results depend on user consistency in creating ROIs; variability between users is acknowledged
- The program was not built to test the accuracy of the user-defined ROIs
- Automated identification of ROIs is beyond the scope of the current version; database synchronization is planned but not implemented
- Over-identification is only mitigated (not eliminated) by the 50% match-ratio threshold; the threshold is customizable by users per study

## Evidence
