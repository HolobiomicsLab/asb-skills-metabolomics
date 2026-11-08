---
name: ion-identity-network-statistics-aggregation
description: Use when you have IIMN (FBMN-IIN) outputs across multiple LC-MS datasets
  and need to summarize and compare the relative abundance of ion species/adducts
  per dataset or sample type.
when_to_use_negative:
- The input is raw spectral data or an unaligned feature list rather than IIMN outputs
  with ion-identity edges already computed.
- The goal is MS2 spectral similarity networking or library matching rather than MS1-based
  ion-species aggregation.
- You need per-feature adduct annotation for a single dataset without cross-dataset
  statistics.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Hosts the FBMN-IIN workflow that produces the IIN supplementary edge files
    aggregated here; the 24 datasets were processed via GNPS.
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Feature-finding tool used to generate the aligned feature lists and MS1-based
    ion identity annotations feeding the aggregation.
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:24e104fc323773db812f9613f5e7a57195abf7044282e35a72e2564870fd1331
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - MZmine
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ion-identity-network-statistics-aggregation

## Summary

Aggregate per-dataset ion-species assignments from MS1-based ion identity networking edges in the IIMN workflow and compute relative ion-formation percentages per adduct (e.g. [M+H]+, [M+Na]+, [M+NH4]+) to compare sample types such as marine vs non-marine datasets.

## When to use

Use when you have IIMN (FBMN-IIN) outputs across multiple LC-MS datasets and need to summarize and compare the relative abundance of ion species/adducts per dataset or sample type.

## When NOT to use

- The input is raw spectral data or an unaligned feature list rather than IIMN outputs with ion-identity edges already computed.
- The goal is MS2 spectral similarity networking or library matching rather than MS1-based ion-species aggregation.
- You need per-feature adduct annotation for a single dataset without cross-dataset statistics.

## Inputs

- Per-dataset ion-identity statistics table from the 24-dataset IIMN processing (MZmine workflow / GNPS FBMN-IIN output)
- IIN supplementary edges files (.csv) linking feature IDs by ion identity
- Feature quantification table (.csv) and MS2 spectral summary (.mgf) exported from the aligned feature list

## Outputs

- Summary table of relative ion-formation percentages per adduct species per dataset (e.g. ion_formation_percentages.csv)
- Group-level mean ± standard deviation per adduct for sample-type comparisons (e.g. marine vs non-marine)

## How to apply

Load the per-dataset ion-identity statistics produced by the 24-dataset MZmine IIMN processing, where edges were formed with a maximum tolerance of 0.001 m/z or 10 ppm. Aggregate the per-molecule ion species assignments from the MS1-based ion identity networking edges (the 'IIN' supplementary edge files) for each dataset. Compute the relative ion-formation percentage of each adduct species per dataset, then group datasets by sample type (e.g. marine vs non-marine) and calculate mean ± standard deviation per adduct group. Validate the aggregation by checking that the computed means fall within the reported ranges, such as marine [M+NH4]+ at 24 ± 5%, overall [M+Na]+ at 26 ± 6%, and overall [M+H]+ at 23 ± 6%. Write the summary table to a CSV file (e.g. ion_formation_percentages.csv) for downstream comparison or reporting.

## Related tools

- **GNPS** (Hosts the FBMN-IIN workflow that produces the IIN supplementary edge files aggregated here; the 24 datasets were processed via GNPS.) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MZmine** (Feature-finding tool used to generate the aligned feature lists and MS1-based ion identity annotations feeding the aggregation.) — https://github.com/mzmine/mzmine

## Examples

```
python scripts/aggregate_ion_formations.py --iin_edges . --out ion_formation_percentages.csv
```

## Evaluation signals

- Computed group means must fall within the reported standard deviations (e.g. marine [M+NH4]+ within 24 ± 5%, overall [M+Na]+ within 26 ± 6%, overall [M+H]+ within 23 ± 6%).
- The output CSV must contain one row per dataset per adduct species with percentage values that sum to ~100% within a dataset.
- Aggregation must only use ion-identity edges from the IIN files that meet the 0.001 m/z / 10 ppm tolerance used to build them.
- Marine vs non-marine group means must reproduce the reported contrast: marine [M+NH4]+ is higher (24 ± 5%) than non-marine (10 ± 8%).

## Limitations

- The 24 datasets used variable feature detection and alignment parameters (summarized in Supplementary Table 4), so cross-dataset comparability depends on the published MZmine workflow rather than identical preprocessing.
- Ion-identity edges are built only within a maximum tolerance of 0.001 m/z or 10 ppm; adducts outside that tolerance will not be captured.
- The IIN edge format is a generic comma-separated text file with columns ID1 and ID2, with no formal schema or validator tool mentioned, so input parsing must handle that format directly.

## Evidence

- [discussion] IIN edge tolerance: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [discussion] export of IIN files: [evidence span withheld — anchor status: not_found]
- [discussion] IIN edge format: "The text format follows a generic comma-separated style with the columns ID1 and ID2"
- [discussion] 24 datasets processed with MZmine workflow: "All 24 datasets (Supplementary Table 1) were processed with the MZmine workflow"
- [intro] IIN layer linking ion identities: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
