---
name: mass-spectrometry-feature-table-statistics
description: Use when a feature quantification table with ion-identity annotations
  is available and you need to test whether specific adduct species (e.g. [M+Na]+,
  [M+NH4]+) change abundance across infusion conditions or sample types.
when_to_use_negative:
- Input is raw MS/MS spectra or an mzML file without a processed, aligned feature
  list — feature detection and IIN annotation must happen first (MZmine/MS-DIAL/XCMS
  → GNPS FBMN-IIN).
- The analysis targets de novo adduct discovery from fragmentation spectra rather
  than statistical validation of already-assigned ion identities.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Produces the aligned feature list and exports the feature quantification table
    plus IIN supplementary edges files that feed the abundance statistics
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Hosts the FBMN-IIN workflow whose output per-dataset ion-identity statistics
    are aggregated for cross-dataset adduct percentage comparisons
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:3460be72ad77d5d91b4c23b1e8bd36ac0e5a7540db5ef8c2a35a194c041b82f2
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mass-spectrometry-feature-table-statistics

## Summary

Statistical comparison of per-adduct feature abundances from MS feature tables (e.g. Welch t-tests on [M+Na]+ / [M+NH4]+ intensities across salt-infusion conditions, and per-dataset adduct percentage summaries) to validate ion-identity assignments.

## When to use

Use when a feature quantification table with ion-identity annotations is available and you need to test whether specific adduct species (e.g. [M+Na]+, [M+NH4]+) change abundance across infusion conditions or sample types.

## When NOT to use

- Input is raw MS/MS spectra or an mzML file without a processed, aligned feature list — feature detection and IIN annotation must happen first (MZmine/MS-DIAL/XCMS → GNPS FBMN-IIN).
- The analysis targets de novo adduct discovery from fragmentation spectra rather than statistical validation of already-assigned ion identities.

## Inputs

- Feature quantification table (.csv) with ion-identity annotations (e.g. MZmine-exported FBMN/IIN feature table)
- MS1-based ion identity network edge file (.csv) with ID1/ID2 columns linking ion species of the same compound
- Sample metadata mapping infusion condition (H2O vs Na-acetate vs NH4-acetate) or sample type (marine vs non-marine)

## Outputs

- Per-adduct Welch t-test results table (mean abundance, fold change, t-statistic, p-value per salt-infusion-vs-control comparison)
- Per-dataset ion-formation percentage summary table (ion_formation_percentages.csv) with marine vs non-marine group means ± standard deviation

## How to apply

Download the deposited Source Data files for Figs. 2–3 accompanying the post-column salt infusion benchmark (300-compound mixture with H2O / Na-acetate / NH4-acetate infusion), parse the ion-identity annotation and per-sample abundance tables, and group adduct abundances by infusion condition. Run Welch two-sample t-tests (unequal variances) comparing [M+Na]+ abundances under Na-acetate infusion against the H2O-infusion control and [M+NH4]+ abundances under NH4-acetate infusion against control, tabulating means, fold changes, t-statistics, and p-values. For cross-dataset summaries, load the per-dataset ion-identity statistics from the 24-dataset MZmine/GNPS FBMN-IIN processing, aggregate per-molecule ion species assigned by MS1-based IIN edges (maximum tolerance 0.001 m/z or 10 ppm), compute per-dataset relative ion-formation percentages, and group datasets by sample type (marine vs non-marine) to report mean ± standard deviation per adduct.

## Related tools

- **MZmine** (Produces the aligned feature list and exports the feature quantification table plus IIN supplementary edges files that feed the abundance statistics) — https://github.com/mzmine/mzmine
- **GNPS** (Hosts the FBMN-IIN workflow whose output per-dataset ion-identity statistics are aggregated for cross-dataset adduct percentage comparisons) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
python run_welch_tests.py --features ion_identity_annotation.csv --conditions salt_infusion_conditions.csv --tests [M+Na]+:Na-acetate_vs_H2O,[M+NH4]+:NH4-acetate_vs_H2O
```

## Evaluation signals

- Reproduced Welch test must show [M+Na]+ and [M+NH4]+ abundances significantly increased over control under the corresponding salt infusion (p < 0.05), matching the reported p < 0.001 significance.
- Marine [M+NH4]+ group mean must fall within 24 ± 5%; non-marine within 10 ± 8%.
- Overall [M+Na]+ percentage must fall within 26 ± 6% and overall [M+H]+ within 23 ± 6% across the 24 datasets.
- Each per-dataset adduct percentage must sum to a plausible relative distribution across all assigned ion species.

## Limitations

- Variable feature detection and alignment parameters for the 24 datasets are only summarized in Supplementary Table 4, not reproducible from text alone.
- No raw data accession ID for the post-column salt infusion experiments is stated in the section text.
- IIN edge format specification (Supplementary Pairs text file) lacks a formal schema or validator tool.

## Evidence

- [results] In the salt infusion benchmark, [M + Na]+ and [M + NH4]+ ion identities were significantly (p < 0.001) more abundant in the sodium acetate or ammonium acetate post-column infusion samples, respectively, compared to controls.: [evidence span withheld — anchor status: not_found]
- [results] Marine samples showed a higher percentage of [M + NH4]+ adducts (24 ± 5%) compared to all other datasets (10 ± 8%): [evidence span withheld — anchor status: not_found]
- [methods] a maximum tolerance of 0.001 m/z or 10 ppm: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [methods] The IIN edge format specification (Supplementary Pairs text file) is described but no formal schema or validator tool is mentioned: "The text format follows a generic comma-separated style with the columns ID1 and ID2"
- [discussion] Perform MS1-based feature grouping and ion identity annotation: "Perform MS1-based feature grouping and ion identity annotation"
