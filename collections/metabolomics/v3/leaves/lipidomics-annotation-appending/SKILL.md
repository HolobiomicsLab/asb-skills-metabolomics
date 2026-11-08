---
name: lipidomics-annotation-appending
description: Use when a feature table (m/z + retention time peaks) contains multiple
  co-eluting lipid candidates with similar m/z values, and you must rank candidate
  identifications and append final lipid annotations to the table.
when_to_use_negative:
- Input lacks MS/MS data (.ms2 files) — ranking depends on experimental fragment intensities,
  not MS1 precursor matching alone
- Vendor data are Waters files, which LipidMatch does not currently support
- The feature table is already fully annotated and ranked by another tool (e.g., MS-DIAL
  or Greazy outputs should be appended afterward, not re-ranked by this step)
tools:
- name: LipidMatch
  role: 'Runs the ranking-and-appending step: ranks candidate lipids per feature by
    summed fragment intensity and appends identifications to each feature table via
    its gWidgets/tcltk dialogue-box UI'
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: R
  role: Language LipidMatch is written in; the appending/ranking logic executes in
    R (pinned to version 3.3.3 in the article)
- name: MZmine
  role: One of the peak-picking tools that produce the feature tables to which ranked
    lipid identifications are appended
- name: MSConvert
  role: Converts vendor MS/MS data to .ms2 format that LipidMatch uses to match fragments
    within the feature's RT and m/z tolerance
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:928d472286f18a04f07e6d302a7c1dc986f206ebfea051195ceaeab68befa899
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - MSConvert
  - MZmine
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# lipidomics-annotation-appending

## Summary

The LipidMatch step that resolves multiple co-eluting lipid candidates per feature by ranking them on summed fragment intensity, then appends the ranked identifications — top-ranked lipid, its class and adduct, and the summed fragment intensity — to the feature table.

## When to use

Use when a feature table (m/z + retention time peaks) contains multiple co-eluting lipid candidates with similar m/z values, and you must rank candidate identifications and append final lipid annotations to the table.

## When NOT to use

- Input lacks MS/MS data (.ms2 files) — ranking depends on experimental fragment intensities, not MS1 precursor matching alone
- Vendor data are Waters files, which LipidMatch does not currently support
- The feature table is already fully annotated and ranked by another tool (e.g., MS-DIAL or Greazy outputs should be appended afterward, not re-ranked by this step)

## Inputs

- Feature table (CSV) from peak-picking software (MZmine, XCMS, MS-DIAL, or Compound Discoverer)
- MS/MS peak-picked .ms2 files (generated with MSConvert) organized in per-feature-table subfolders
- In silico lipid fragment libraries in .csv format (LipidMatch distribution)
- Per-lipid-class fragment statistics tables from fragment matching (Step 3 of Fig. 3)

## Outputs

- Feature table with appended ranked lipid identifications per feature
- Lipid class and adduct of the top-ranked candidate per feature
- Summed fragment intensity of each candidate identification

## How to apply

Load the per-lipid-class CSV fragment statistics tables generated during fragment matching (Step 3 of the LipidMatch workflow, Fig. 3), then for each candidate lipid of a feature collect all experimental fragments with exact-mass matches to in silico fragment m/z values — including fragments not required for confirmatory identification. Sum fragment intensities per candidate using the scan with the highest intensity for each fragment across the feature peak, and rank candidate lipids from maximum to minimum summed fragment intensity. Append the ranked lipid identifications to the feature table together with the lipid class and adduct of the top-ranked lipid and the summed fragment intensity of each identification. Rationale: summed intensity uses all corroborating fragment evidence, so co-eluting isomers with similar precursor m/z but distinct fragments can be disambiguated beyond the class/fatty-acyl rules applied in confirmation steps (Steps 5–6). Verify the ranking output against the example output data included in the LipidMatch distribution (Additional file 1, secim.ufl.edu/secim-tools/).

## Related tools

- **LipidMatch** (Runs the ranking-and-appending step: ranks candidate lipids per feature by summed fragment intensity and appends identifications to each feature table via its gWidgets/tcltk dialogue-box UI) — https://github.com/GarrettLab-UF/LipidMatch
- **R** (Language LipidMatch is written in; the appending/ranking logic executes in R (pinned to version 3.3.3 in the article))
- **MZmine** (One of the peak-picking tools that produce the feature tables to which ranked lipid identifications are appended)
- **MSConvert** (Converts vendor MS/MS data to .ms2 format that LipidMatch uses to match fragments within the feature's RT and m/z tolerance)

## Evaluation signals

- Each feature with multiple candidates receives a complete ranked list ordered from maximum to minimum summed fragment intensity
- Summed intensity is computed over all fragments with exact-mass matches to in silico fragment m/z, including fragments not required for confirmatory identification (Step 4/5 fragments excluded from the rule check are still summed)
- Per fragment, the intensity used is from the scan with the highest intensity across the feature peak
- Appended columns include the top-ranked lipid's class and adduct alongside the summed fragment intensity
- Output matches the example ranking data in Additional file 1 of the LipidMatch distribution

## Limitations

- Ranking is by summed fragment intensity only; co-eluting candidates with identical fatty acyl compositions and no distinguishing matched fragments cannot be resolved
- Software was only tested on Windows 7 through 10 and pinned to R version 3.3.3; no cross-platform support stated
- Key artifacts (software zip, lipid libraries, example outputs) are only provided as journal additional files, not archived in a public repository
- Waters vendor files are not currently supported (README)

## Evidence

- [methods] the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3).: "the lipids are ranked by the summed intensity of all their fragments with in silico fragment exact mass matches, including those not used for confirmation (Step 7, Fig. 3)."
- [abstract] Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities.: "Another unique feature of LipidMatch is ranking lipid identifications for a given feature by the sum of fragment intensities."
- [methods] Once the user runs LipidMatch and enters user parameters, LipidMatch will automatically append identifications to each feature table using MS/MS files contained in that feature table’s subfolder.: "Once the user runs LipidMatch and enters user parameters, LipidMatch will automatically append identifications to each feature table using MS/MS files contained in that feature table’s subfolder."
- [readme] LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software.: "LipidMatch can be used with various peak picking software (for example MZmine, XCMS, MS-DIAL, and Compound Discoverer), and combine results from other lipidomics software."
