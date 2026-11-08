---
name: marine-vs-non-marine-sample-group-contrast
description: Use when multiple LC-MS/MS datasets have been processed with the IIMN
  workflow (MZmine feature lists + GNPS FBMN-IIN) and you need to compare relative
  adduct-formation percentages between sample-type groups such as marine vs non-marine
  samples.
when_to_use_negative:
- The input is raw LC-MS/MS spectra or unprocessed feature lists — ion species assignments
  must already exist from MZmine feature grouping plus GNPS FBMN-IIN before group
  contrast is meaningful.
- Only one dataset is available with no sample-type grouping — group mean ± standard
  deviation contrasts require multiple datasets per group.
- Dataset metadata lacks sample-type labels (marine/non-marine), so the grouping step
  of the contrast cannot be defined.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine
  role: Feature finding, MS1-based feature grouping and ion identity annotation that
    produce the per-dataset ion species assignments aggregated in this skill
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Runs the FBMN-IIN workflow on the web server, producing the ion identity networking
    edges and ion-identity statistics consumed for adduct percentage computation
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:680eaecb9d14da8058fbe40882a6d8b88ecd2525dc1d56022f47b591074cde4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marine-vs-non-marine-sample-group-contrast

## Summary

Aggregates per-molecule ion species (adduct) assignments from IIMN-processed datasets into relative ion-formation percentages, then contrasts marine vs non-marine sample groups (e.g., [M+NH4]+ at 24±5% vs 10±8%). Provides a validated group-level comparison of MS1-based ion identity networking results across multiple public datasets.

## When to use

Use when multiple LC-MS/MS datasets have been processed with the IIMN workflow (MZmine feature lists + GNPS FBMN-IIN) and you need to compare relative adduct-formation percentages between sample-type groups such as marine vs non-marine samples.

## When NOT to use

- The input is raw LC-MS/MS spectra or unprocessed feature lists — ion species assignments must already exist from MZmine feature grouping plus GNPS FBMN-IIN before group contrast is meaningful.
- Only one dataset is available with no sample-type grouping — group mean ± standard deviation contrasts require multiple datasets per group.
- Dataset metadata lacks sample-type labels (marine/non-marine), so the grouping step of the contrast cannot be defined.

## Inputs

- Per-dataset ion-identity statistics table from the 24-dataset IIMN processing (GNPS FBMN-IIN output)
- IIN supplementary edges files (.csv) linking ion identities of the same compound at 0.001 m/z / 10 ppm tolerance
- Feature quantification table (.csv) and MS2 spectral summary (.mgf) exported from MZmine
- Dataset metadata mapping each of the 24 public experimental datasets to a sample type (marine vs non-marine)

## Outputs

- ion_formation_percentages.csv — summary table of relative ion-formation percentages per adduct species per dataset, with marine vs non-marine group means and standard deviations
- Validation report of computed group means against reported anchors ([M+NH4]+ 24±5% marine vs 10±8% non-marine, [M+Na]+ 26±6% overall, [M+H]+ 23±6% overall)

## How to apply

Load the per-dataset ion-identity statistics produced by the 24-dataset IIMN processing (MZmine workflow / GNPS FBMN-IIN output), where ion species were linked by MS1-based ion identity networking edges computed at a maximum tolerance of 0.001 m/z or 10 ppm. Aggregate per-molecule ion species assignments per dataset, then compute the relative ion-formation percentage of each adduct species per dataset. Group datasets by sample type (marine vs non-marine) and compute mean ± standard deviation per adduct group, plus overall means for protonated and sodiated species. Validate the reproduction against reported anchors: marine [M+NH4]+ within 24±5%, overall [M+Na]+ within 26±6%, overall [M+H]+ within 23±6%. Write the summary table to ion_formation_percentages.csv.

## Related tools

- **MZmine** (Feature finding, MS1-based feature grouping and ion identity annotation that produce the per-dataset ion species assignments aggregated in this skill) — https://github.com/mzmine/mzmine
- **GNPS** (Runs the FBMN-IIN workflow on the web server, producing the ion identity networking edges and ion-identity statistics consumed for adduct percentage computation) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
import pandas as pd; df = pd.read_csv("iin_edges.csv"); df.groupby(["dataset","adduct"]).size().pipe(lambda s: s/s.groupby("dataset").transform("sum")*100).to_csv("ion_formation_percentages.csv")
```

## Evaluation signals

- Computed marine [M+NH4]+ mean falls within the reported 24±5% band and non-marine within 10±8%.
- Computed overall [M+Na]+ mean falls within 26±6% and overall [M+H]+ within 23±6%.
- Relative ion-formation percentages sum to 100% per dataset when all adduct species are included.
- ion_formation_percentages.csv contains one row per dataset per adduct group plus marine/non-marine group summary rows with mean and standard deviation.
- Adduct assignments trace back to IIN edges that respected the 0.001 m/z or 10 ppm maximum tolerance.

## Limitations

- The 24 public experimental datasets were processed with variable feature detection and alignment parameters summarized only in Supplementary Table 4, so exact reproduction of per-dataset preprocessing is not possible from the text alone.
- Group contrast depends on the marine/non-marine labeling of public datasets; mislabeled or heterogeneous metadata will shift the reported adduct percentages.
- Ion species statistics depend on the MS1-based IIN edge tolerances (0.001 m/z or 10 ppm) and Pearson correlation thresholds (feature shape ≥ 0.85 with ≥ 5 data points; feature height ≥ 0.6 with ≥ 3 data points); different settings change adduct percentages.

## Evidence

- [intro] 24 public experimental datasets presented with IIMN results: "we present IIMN results for two datasets of natural products standards as well as 24 publicly available experimental datasets"
- [discussion] Maximum ion identity networking tolerance: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [discussion] Feature grouping and ion identity annotation step: "Perform MS1-based feature grouping and ion identity annotation"
- [discussion] Export of IIN edges and feature tables consumed by this skill: [evidence span withheld — anchor status: not_found]
- [readme] MZmine as the upstream feature-processing tool: "mzmine is an open-source software for mass spectrometry data processing. The goals of the project is
to provide a user-friendly, flexible and easily extendable software with a complete set of"
