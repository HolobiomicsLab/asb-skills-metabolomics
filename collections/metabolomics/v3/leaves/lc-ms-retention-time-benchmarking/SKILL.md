---
name: lc-ms-retention-time-benchmarking
description: Use when you have metabolite standard peak lists from both a manual vendor
  tool (e.g. MassHunter) and a semiautomated RT assignment pipeline (e.g. metScribeR)
  and need to quantify RT assignment accuracy for library quality control.
when_to_use_negative:
- Input is already a feature table from targeted peak picking tools such as MZmine
  or MS-DIAL rather than standards mzML files needing RT assignment
- You lack a manual (vendor-tool) RT reference set to compare against, since the skill's
  core output is a benchmark correlation/RMSD
- The standards data require MS/MS-based library construction — metScribeR does not
  use MS2 data for library building
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: R package with Shiny app used to generate the semiautomated RT assignments
    being benchmarked (create_eic, find_eic_peaks, export_library_csv)
  repo: https://github.com/ncats/metScribeR
- name: chromatographR
  role: dependency installed for peak/EIC handling within the metScribeR workflow
  repo: https://github.com/ethanbass/chromatographR
- name: mzR
  role: Bioconductor package used by metScribeR to parse mzML files into EICs
- name: MassHunter
  role: Vendor tool providing the manual (gold-standard) RT assignments used as the
    reference in the accuracy comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:083d780a057236664400add4834370625468e018eebc69321ac22c86ca0c13e8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - MassHunter
  - R
  - Shiny
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lc-ms-retention-time-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for validating semiautomated LC-MS retention-time assignment against a manual reference: it joins metScribeR-derived RTs to manually annotated MassHunter RTs for authentic IROA standards and computes Pearson correlation and root-mean-square deviation (RMSD) to quantify agreement.

## When to use

Use when you have metabolite standard peak lists from both a manual vendor tool (e.g. MassHunter) and a semiautomated RT assignment pipeline (e.g. metScribeR) and need to quantify RT assignment accuracy for library quality control.

## When NOT to use

- Input is already a feature table from targeted peak picking tools such as MZmine or MS-DIAL rather than standards mzML files needing RT assignment
- You lack a manual (vendor-tool) RT reference set to compare against, since the skill's core output is a benchmark correlation/RMSD
- The standards data require MS/MS-based library construction — metScribeR does not use MS2 data for library building

## Inputs

- standards_df.csv with common_name, monoisotopic_mass, pos_mode_mzML_file_path, neg_mode_mzML_file_path columns
- adduct_df.csv with adduct, change_from_neutral, mode columns
- Positive and negative ESI mzML files of authentic standards (e.g. IROA metabolite standards)
- Manual MassHunter RT assignments for the same standards

## Outputs

- rt_accuracy_metrics.csv containing the Pearson correlation and RMSD of manual vs metScribeR RTs
- exported_metScribeR_library.csv with assigned RTs, m/z, and peak quality metrics
- exported_metScribeR_library_with_metrics.csv including all 'good', 'bad', and 'multimodal/indeterminate' adducts

## How to apply

Run metScribeR on the benchmark standards: create EICs for all standard adducts (create_eic/import_eic_data) from pos and neg ESI mzML files using an mzML parsing noise level of 1000 and user-defined m/z and RT tolerances; detect peaks with find_eic_peaks; apply the benchmark filter criteria (≥7 data points per peak, width at half maximum intensity >0 s and <15 s, ≤250 peaks per EIC, and intensity ≥10,000 i.e. 10× the background noise level); then resolve best adducts with calc_peak_metrics, add_adduct_peak_presence, and update_best_peak_and_adduct, and export RTs via export_library_csv. Join the metScribeR-derived RTs (exported_metScribeR_library.csv) to the manual MassHunter RTs on common_name for the IROA metabolite standards and compute the Pearson correlation coefficient and the RMSD of the RT assignments, writing both values to rt_accuracy_metrics.csv. The article's benchmark reports a Pearson correlation of 0.99 and an RMSD of 0.54 s — equivalent to 0.18% of the 5 min column cycle time — so accept the recomputation if these metrics match within reasonable numerical tolerance.

## Related tools

- **metScribeR** (R package with Shiny app used to generate the semiautomated RT assignments being benchmarked (create_eic, find_eic_peaks, export_library_csv)) — https://github.com/ncats/metScribeR
- **chromatographR** (dependency installed for peak/EIC handling within the metScribeR workflow) — https://github.com/ethanbass/chromatographR
- **mzR** (Bioconductor package used by metScribeR to parse mzML files into EICs)
- **MassHunter** (Vendor tool providing the manual (gold-standard) RT assignments used as the reference in the accuracy comparison)

## Examples

```
library(metScribeR); setwd('metScribeR_extdata_folder'); runMetScribeRShinyApp()
```

## Evaluation signals

- Pearson correlation of manual vs metScribeR RTs matches the reported 0.99 within numerical tolerance
- RT RMSD matches the reported 0.54 s (equivalent to 0.18% of the 5 min column cycle time)
- exported_metScribeR_library.csv contains only peaks that passed all filter criteria: ≥7 data points, width at half maximum >0 s and <15 s, ≤250 peaks per EIC, intensity ≥10,000
- Each reviewed peak has an explicit manual quality label (Good Peak / Bad Peak / Multimodal/Indeterminate) recorded in the with_metrics export

## Limitations

- MS/MS collection from MoNA is dependent on server speed and can be slow, so the benchmark may stall on large libraries
- The initial submit step for large submissions takes substantial time (~30 min for 12,000 mzML files)
- Width-at-half-maximum and RT-tolerance cutoffs must be tuned to the specific LC-MS method; the benchmark values (e.g. <15 s FWHM) may not generalize to faster or slower gradients
- metScribeR reports an identification probability estimate per adduct but does not require MS/MS spectral information, so its RT accuracy metric does not validate spectral matching

## Evidence

- [results] The Pearson correlation between manual and metScribeR RTs was 0.99, with an RMSD of 0.54 s, equivalent to 0.18% of the 5 min column cycle time.: [evidence span withheld — anchor status: not_found]
