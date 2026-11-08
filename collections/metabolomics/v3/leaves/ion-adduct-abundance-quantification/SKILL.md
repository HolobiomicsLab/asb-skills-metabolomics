---
name: ion-adduct-abundance-quantification
description: Use when you have per-dataset IIMN/FBMN-IIN output (feature quantification
  table plus IIN edges CSVs) from MZmine-processed LC-MS/MS datasets and need relative
  adduct percentages per dataset or per sample-type group.
when_to_use_negative:
- Input is raw LC-MS/MS data or an unprocessed feature table that has not yet gone
  through MS1-based feature grouping and ion identity annotation — first run the IIMN
  workflow
- Analysis relies only on MS2 spectral similarity networks without MS1 ion-identity
  edges, since adduct species assignments come from the MS1-based IIN layer
- Single-dataset analysis with no sample-type grouping, where per-dataset adduct percentages
  cannot be contrasted between groups
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine
  role: Feature finding, aligned feature list generation, and MS1-based feature grouping
    / ion identity annotation that produces the IIN edges the adduct statistics are
    computed from
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Hosts the FBMN-IIN workflow that fuses MS1-based ion identity networks with
    MS2-based feature molecular networking; consumes the uploaded feature table, .mgf,
    and IIN edges files
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: xcms-gnps-tools
  role: Provides getFeatureAnnotations() to extract adduct annotations in the IIN
    format required for FBMN/GNPS when using XCMS instead of MZmine
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:75513de87138408756f374021b74e35be4ae8cbd97841798c7718563eb8ac5a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - xcms-gnps-tools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ion-adduct-abundance-quantification

## Summary

Quantifies the relative ion-formation (adduct) percentages of [M+H]+, [M+Na]+, [M+NH4]+ and related species from MS1-based ion identity networking (IIN) edges produced by the IIMN/FBMN workflow, and compares them across datasets grouped by sample type (e.g., marine vs non-marine).

## When to use

Use when you have per-dataset IIMN/FBMN-IIN output (feature quantification table plus IIN edges CSVs) from MZmine-processed LC-MS/MS datasets and need relative adduct percentages per dataset or per sample-type group.

## When NOT to use

- Input is raw LC-MS/MS data or an unprocessed feature table that has not yet gone through MS1-based feature grouping and ion identity annotation — first run the IIMN workflow
- Analysis relies only on MS2 spectral similarity networks without MS1 ion-identity edges, since adduct species assignments come from the MS1-based IIN layer
- Single-dataset analysis with no sample-type grouping, where per-dataset adduct percentages cannot be contrasted between groups

## Inputs

- Per-dataset ion-identity statistics table from IIMN processing (MZmine workflow / GNPS FBMN-IIN output)
- IIN supplementary edges files (.csv pairs files with ID1/ID2 columns) from MS1-based ion identity networking
- Feature quantification table (.csv) exported from MZmine
- MS2 spectral summary file (.mgf) exported alongside the feature table
- Dataset metadata assigning each of the 24 public datasets to a sample type (marine vs non-marine)

## Outputs

- ion_formation_percentages.csv containing per-dataset relative adduct percentages per ion species
- Group-level mean ± standard deviation per adduct for marine vs non-marine datasets
- Overall means for protonated ([M+H]+ 23 ± 6%), sodiated ([M+Na]+ 26 ± 6%), and ammoniated species
- Validation comparison of computed group means against the reported values

## How to apply

Load the per-dataset ion-identity statistics from the MZmine/GNPS FBMN-IIN output, then aggregate per-molecule ion-species assignments derived from MS1-based ion identity networking edges computed at a maximum tolerance of 0.001 m/z or 10 ppm. For each dataset, compute the relative ion-formation percentage of each adduct species from the aggregated assignments. Group datasets by sample type (marine vs non-marine) and compute mean ± standard deviation per adduct group alongside overall means for protonated and sodiated species. Validate group means against the reported contrasts: marine [M+NH4]+ at 24 ± 5% versus 10 ± 8% in all other datasets, overall [M+Na]+ at 26 ± 6%, and overall [M+H]+ at 23 ± 6%. Write the final summary table to ion_formation_percentages.csv. Note the 24 public datasets were processed with the MZmine workflow with variable feature-detection and alignment parameters summarized in Supplementary Table 4.

## Related tools

- **MZmine** (Feature finding, aligned feature list generation, and MS1-based feature grouping / ion identity annotation that produces the IIN edges the adduct statistics are computed from) — https://github.com/mzmine/mzmine
- **GNPS** (Hosts the FBMN-IIN workflow that fuses MS1-based ion identity networks with MS2-based feature molecular networking; consumes the uploaded feature table, .mgf, and IIN edges files) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **xcms-gnps-tools** (Provides getFeatureAnnotations() to extract adduct annotations in the IIN format required for FBMN/GNPS when using XCMS instead of MZmine) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
import pandas as pd; t = pd.read_csv('iin_edges.csv'); g = t.groupby(['dataset', 'ion_species']).size(); (g / g.groupby('dataset').transform('sum') * 100).to_csv('ion_formation_percentages.csv')
```

## Evaluation signals

- Computed marine [M+NH4]+ mean falls within the reported 24 ± 5% (vs 10 ± 8% for non-marine datasets)
- Computed overall [M+Na]+ percentage falls within the reported 26 ± 6%
- Computed overall [M+H]+ percentage falls within the reported 23 ± 6%
- Per-dataset adduct percentages sum to ~100% of aggregated ion-species assignments
- IIN edges underlying the counts respect the maximum tolerance of 0.001 m/z or 10 ppm

## Limitations

- Variable feature-detection and alignment parameters for the 24 datasets are only summarized in Supplementary Table 4 and are not reproducible from the text alone
- The IIN edges file follows a generic comma-separated format (ID1, ID2 columns) without a formal schema or validator tool
- The MZmine IIMN build is described as a 'special IIMN build' and it is unclear whether it is part of a released MZmine version

## Evidence

- [intro] we present IIMN results for two datasets of natural products standards as well as 24 publicly available experimental datasets: "we present IIMN results for two datasets of natural products standards as well as 24 publicly available experimental datasets"
- [discussion] IIN tolerance defines which MS1 edges count as ion-identity links: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [discussion] All 24 datasets were processed with the MZmine workflow: "All 24 datasets (Supplementary Table 1) were processed with the MZmine workflow"
- [discussion] The workflow requires MS1-based feature grouping and ion identity annotation before statistics can be extracted: "Perform MS1-based feature grouping and ion identity annotation"
- [readme] xcms-gnps-tools provides adduct annotations in the IIN format required by FBMN/GNPS: [evidence span withheld — anchor status: not_found]
