---
name: blank-sample-filtering
description: Use when running the UmetaFlow pre-processing step on LC-MS/MS mzML data
  that includes blank or QC injections (e.g. ISP2_blank media controls or SA/SB Piper
  nigrum mixture replicates in MTBLS733) and contaminants from media/blank carryover
  must be excluded before feature quantification.
when_to_use_negative:
- The dataset contains no blank or QC injections (skip the blank/QC-removal step;
  set the flag FALSE in config.yaml).
- Input is already a cleaned feature quantification table rather than raw/centroided
  mzML data — pre-processing and blank removal do not apply.
- Vendor files are Agilent or Bruker and were not placed in data/mzML/ — the file-conversion
  rule only handles Thermo raw files, so convert externally with proteowizard first.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: UmetaFlow
  role: Workflow (Snakemake and pyOpenMS/Jupyter implementations) whose pre-processing
    step performs the optional removal of blank/QC samples and the minfrac missing-value
    filter
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: Snakemake
  role: Executes the workflow; blank/QC removal and minfrac are toggled via TRUE/FALSE
    flags in config.yaml
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: OpenMS
  role: Provides the underlying feature detection, map alignment and feature grouping
    algorithms on which the blank/QC filtering is applied
- name: pyOpenMS
  role: Python bindings to OpenMS algorithms used in the Jupyter notebook implementation
    of the same pre-processing/filtering step
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:77f69382da77d5abd8c7136aa49a20181025914fb0ee2dbe5f6dc0152bd568a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - Jupyter
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# blank-sample-filtering

## Summary

An optional UmetaFlow pre-processing sub-step that removes blank/QC samples (and, via the minfrac option, consensus features with too many missing values) so downstream feature detection, alignment and grouping reflect true biological samples.

## When to use

Use when running the UmetaFlow pre-processing step on LC-MS/MS mzML data that includes blank or QC injections (e.g. ISP2_blank media controls or SA/SB Piper nigrum mixture replicates in MTBLS733) and contaminants from media/blank carryover must be excluded before feature quantification.

## When NOT to use

- The dataset contains no blank or QC injections (skip the blank/QC-removal step; set the flag FALSE in config.yaml).
- Input is already a cleaned feature quantification table rather than raw/centroided mzML data — pre-processing and blank removal do not apply.
- Vendor files are Agilent or Bruker and were not placed in data/mzML/ — the file-conversion rule only handles Thermo raw files, so convert externally with proteowizard first.

## Inputs

- Centroided mzML files in data/mzML/ (converted from Thermo raw via ThermoRawFileParser, or Agilent/Bruker via proteowizard)
- config/dataset.tsv listing sample_name and comment for all files to process
- config/blanks.tsv declaring blank/QC sample names (e.g. ISP2_blank)
- config/samples.tsv declaring biological sample names
- config.yaml with pre-processing/re-quantification step flags (TRUE/FALSE) and ionisation parameters

## Outputs

- Blank/QC-filtered consensus feature table from OpenMS feature detection, alignment and grouping
- minfrac-filtered feature quantification matrix (optionally after re-quantification)
- Final re-quantified feature table (TSV) with missing values recovered

## How to apply

Declare blank and QC injections in config/blanks.tsv (e.g. 'ISP2_blank | blank media') and true biological samples in config/samples.tsv / config/dataset.tsv, then set the blank/QC-removal flag in config.yaml when configuring the Snakemake workflow (or run the equivalent cell in the pyOpenMS notebook). The removal is applied during step 2 (pre-processing: OpenMS feature detection, map alignment and feature grouping). Optionally enable the 'minfrac' step after pre-processing and after re-quantification to drop consensus features with too many missing values. The output is a cleaned feature quantification matrix used for statistical analysis; benchmark on MTBLS733 by comparing detected features against the 836 targeted ground-truth features (expected ~778 detected, 736 accurately quantified).

## Related tools

- **UmetaFlow** (Workflow (Snakemake and pyOpenMS/Jupyter implementations) whose pre-processing step performs the optional removal of blank/QC samples and the minfrac missing-value filter) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **Snakemake** (Executes the workflow; blank/QC removal and minfrac are toggled via TRUE/FALSE flags in config.yaml) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **OpenMS** (Provides the underlying feature detection, map alignment and feature grouping algorithms on which the blank/QC filtering is applied)
- **pyOpenMS** (Python bindings to OpenMS algorithms used in the Jupyter notebook implementation of the same pre-processing/filtering step) — https://github.com/biosustain/pyOpenMS_UmetaFlow.git

## Examples

```
snakemake --use-conda --cores all  # with blank/QC removal and minfrac flags set TRUE in config.yaml
```

## Evaluation signals

- config/blanks.tsv and config/samples.tsv correctly declare every blank/QC and biological sample present in data/raw/ or data/mzML/.
- After filtering, no blank sample (e.g. ISP2_blank) appears in the feature quantification matrix sample columns.
- The minfrac option removes consensus features with too many missing values; enabling re-quantification afterwards should recover features with missing values across samples.
- On MTBLS733, the filtered pipeline should detect ~778 of 836 targeted ground-truth features (93.1%) and accurately quantify 736 (94.6%).
- A snakemake --use-conda --dry-run completes without errors, confirming the config.yaml step flags are set consistently.

## Limitations

- Blank/QC removal is optional and only applied 'if defined by the user' — forgetting to declare blanks in config/blanks.tsv silently leaves contaminants in the feature table.
- The minfrac filter removes consensus features with too many missing values and may drop genuine low-prevalence features; it is applied separately after pre-processing and after re-quantification.
- The workflow (Snakemake implementation) targets Linux and MacOS; macOS on Silicon requires a manual OpenMS 3.3.0 install and codesign step.
- Benchmark performance numbers (e.g. Tables S5, S7 for MTBLS733) are only in supplementary files, and no software version numbers are stated for UmetaFlow/OpenMS/Snakemake in the main text.

## Evidence

- [methods] Optional blank/QC removal: "This step includes an optional removal of blank/QC samples if defined by the user."
- [methods] minfrac after pre-processing: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] minfrac after re-quantification: "Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration."
- [abstract] MTBLS733 benchmark performance: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
- [readme] blank declaration format: "If there are blanks/QC samples in the file list, add them to the appropriate table."
