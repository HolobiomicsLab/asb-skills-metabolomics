---
name: msms-scan-statistics-aggregation
description: Use when a feature table row has multiple lipid candidates passing rule-based
  identification (same or similar m/z, co-eluting) and you must rank them by summed
  fragment intensity across MS/MS scans spanning the feature peak.
when_to_use_negative:
- A feature has only one candidate lipid passing confirmatory fragment requirements
  — ranking adds no information
- You need identification confirmation itself (required-fragment/scan-count binarization,
  Steps 4–5) rather than candidate ordering
- Waters vendor files are the input, since LipidMatch does not currently support Waters
  files
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LipidMatch
  role: Performs the summed-fragment-intensity ranking (Step 7) and appends ranked
    identifications, lipid class, and adduct to feature tables
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: R
  role: Language in which LipidMatch implements the fragment matching, scan statistics,
    and intensity summation; also used to generate the in silico libraries
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:b93eca2e0758728276dd13fe230b7f0b76c1a7b50144b71cd6a131b0941aab3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# MS/MS scan statistics aggregation for summed-fragment-intensity ranking

## Summary

Aggregates per-scan fragment match statistics into a summed fragment intensity score that ranks multiple lipid candidates co-eluting within a single feature (defined by m/z and retention time). This is LipidMatch's unique disambiguation mechanism for features containing multiple co-eluting molecules with similar m/z values.

## When to use

Use when a feature table row has multiple lipid candidates passing rule-based identification (same or similar m/z, co-eluting) and you must rank them by summed fragment intensity across MS/MS scans spanning the feature peak.

## When NOT to use

- A feature has only one candidate lipid passing confirmatory fragment requirements — ranking adds no information
- You need identification confirmation itself (required-fragment/scan-count binarization, Steps 4–5) rather than candidate ordering
- Waters vendor files are the input, since LipidMatch does not currently support Waters files

## Inputs

- Per-lipid-class CSV fragment statistics tables from LipidMatch fragment matching (Step 3), one per feature-lipid pair
- MS/MS scans from .ms2 files within user-defined retention time and m/z tolerance of each feature
- Feature table with multiple lipid candidate identifications per feature (from MZmine, XCMS, or MS-DIAL peak picking)
- In silico lipid fragment m/z libraries (CSV format)

## Outputs

- Ranked lipid identification list per feature, ordered from maximum to minimum summed fragment intensity
- Feature table annotated with lipid class and adduct of the top-ranked lipid plus the summed fragment intensity of each identification

## How to apply

Load the per-lipid-class CSV fragment statistics tables generated during fragment matching (Step 3 of the LipidMatch workflow), which record, for each experimental fragment matched within a ppm tolerance of in silico lipid fragment m/z values, the total number of scans across the feature containing that fragment. For each candidate lipid, collect all experimental fragments with exact-mass matches to in silico fragment m/z values — deliberately including fragments that are not required for confirmatory identification — and take the highest-intensity scan for each fragment across the feature peak. Sum these fragment intensities per candidate (Steps 4 and 7 of the Fig. 3 workflow, after fragments are binarized 1/0 against user-defined minimum intensity and scan-count thresholds). Rank candidates from maximum to minimum summed fragment intensity, then append the ranked identifications to the feature table together with the lipid class, adduct of the top-ranked lipid, and the summed fragment intensity of each identification.

## Related tools

- **LipidMatch** (Performs the summed-fragment-intensity ranking (Step 7) and appends ranked identifications, lipid class, and adduct to feature tables) — https://github.com/GarrettLab-UF/LipidMatch
- **R** (Language in which LipidMatch implements the fragment matching, scan statistics, and intensity summation; also used to generate the in silico libraries)

## Evaluation signals

- Every fragment contributing to the sum has an exact-mass match to an in silico fragment m/z within the stated ppm tolerance window
- For each fragment, the intensity used comes from the scan with the highest intensity for that fragment across the feature peak (not summed across all scans)
- Fragments failing the user-defined minimum intensity and scans threshold are excluded from confirmation yet still included in the intensity sum, matching Step 7 semantics
- Final output appends lipid class and adduct of the top-ranked lipid alongside each summed fragment intensity
- Ranking output matches the example output data included in the LipidMatch distribution (Additional file 1)

## Limitations

- Ranking is by summed fragment intensity only; a co-eluting lower-abundance isomer with diagnostic fragments may still rank below a more intense interferent
- Fragments not required for confirmatory identification contribute to the score, so intensities from non-diagnostic fragments can influence the ranking
- Fragment statistics depend on ddMS2-topN acquisition quality; iterative exclusion (IE) on pooled samples may be needed to obtain fragmentation spectra for more ions
- Requires .ms2 files generated externally via MSConvert and feature tables from external peak picking software

## Evidence

- [abstract] Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities.: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [methods] The total number of scans across a feature containing that fragment is calculated.: "experimental fragments are matched against in silico lipid fragments m/z using a tolerance window (ppm). The total number of scans across a feature containing that fragment is calculated."
- [methods] Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria: "Each fragment is assigned 1 if it is above the user defined minimum intensity and scans threshold and 0 if the fragment does not meet these criteria or was not found within the m/z tolerance (Step 4;"
- [methods] the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3)."
- [intro] features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values: "features (peaks defined by a mass to charge ratio (m/z) and retention time) often contain multiple co-eluting molecules with similar m/z values"
