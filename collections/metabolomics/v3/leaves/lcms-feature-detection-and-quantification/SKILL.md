---
name: lcms-feature-detection-and-quantification
description: Use when raw LC-MS/MS untargeted metabolomics data (Thermo RAW, Agilent,
  Bruker, or mzML files) must be converted into a table of metabolic features with
  alignment, adduct annotation, and quantification, and you need benchmarkable detection/quantification
  rates.
when_to_use_negative:
- Input is already a processed feature table or annotated feature matrix — skip directly
  to re-quantification or downstream statistics rather than re-running pre-processing.
- Data are already deconvoluted and aligned, and you only need marker selection or
  SIRIUS/GNPS annotation — the pre-processing feature detection stage would duplicate
  work already done.
- Working with non-metabolomics data (e.g. DNA-seq variant calling), where the analogous
  snakemake-workflows/dna-seq-gatk-variant-calling pipeline is the appropriate reference.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: UmetaFlow
  role: End-to-end workflow that orchestrates file conversion, feature detection,
    alignment, adduct annotation, re-quantification and blank/minfrac filtering
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS 3.0
  role: Underlying tools for feature detection, map alignment, adduct annotation,
    re-quantification and feature linking invoked via the Snakemake rules
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Snakemake
  role: Workflow engine running the pre-processing and re-quantification steps with
    per-rule conda environments
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: pyOpenMS
  role: Python bindings to the OpenMS algorithms used in the Jupyter-notebook implementation
    of the same pre-processing pipeline
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: ThermoRawFileParser
  role: Converts Thermo RAW files into centroided mzML in the file conversion step
    (referenced in the MacOS setup script)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:795e1e6d6ccace92d0fb5a55605f29eb16796de5344823959bc756ea27198fbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - Jupyter
  - OpenMS
  - OpenMS 3.0
  - Python
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# LC-MS feature detection and quantification

## Summary

UmetaFlow's OpenMS-based pre-processing pipeline converts centroided mzML LC-MS/MS data into an annotated feature table (mzML → featureXML → consensus features with adduct/linking annotations), then optionally re-quantifies features with missing values across samples. On the MTBLS736 benchmark (AB SCIEX TripleTOF 6600), it detected 90.1% of ground truth features and accurately quantified 81.7% of them.

## When to use

Use when raw LC-MS/MS untargeted metabolomics data (Thermo RAW, Agilent, Bruker, or mzML files) must be converted into a table of metabolic features with alignment, adduct annotation, and quantification, and you need benchmarkable detection/quantification rates.

## When NOT to use

- Input is already a processed feature table or annotated feature matrix — skip directly to re-quantification or downstream statistics rather than re-running pre-processing.
- Data are already deconvoluted and aligned, and you only need marker selection or SIRIUS/GNPS annotation — the pre-processing feature detection stage would duplicate work already done.
- Working with non-metabolomics data (e.g. DNA-seq variant calling), where the analogous snakemake-workflows/dna-seq-gatk-variant-calling pipeline is the appropriate reference.

## Inputs

- Thermo RAW files in data/raw/ (converted to centroided mzML via file conversion rule)
- Agilent or Bruker files converted independently with ProteoWizard, placed in data/mzML/
- config/dataset.tsv sample table (generated with 'python data_files.py')
- config/blanks.tsv and config/samples.tsv if blank/QC samples must be removed
- config/config.yaml with dataset-specific ionisation mode (positive/negative) and other parameters

## Outputs

- Centroided mzML files (results after file conversion)
- Feature tables in TSV format (results/interim) with feature detection, map alignment, adduct annotation and feature linking applied
- Re-quantified feature matrix with missing values recovered (optional step)
- Annotated feature matrix with quantification and adduct/linking annotations

## How to apply

Run the UmetaFlow pre-processing step: place Thermo RAW files in data/raw/ (or Agilent/Bruker-converted centroid mzML in data/mzML/), generate config/dataset.tsv via 'python data_files.py', set ionisation mode in config/config.yaml, and execute 'snakemake --use-conda --cores all'. OpenMS 3.0 tools perform feature detection, map alignment, adduct annotation, feature linking, and an optional blank/QC removal and 'minfrac' filtering of consensus features with too many missing values. Then run the optional re-quantification step to recover features with missing values across samples, which improves statistical analysis and data exploration. Validate against MTBLS736 ground truth: accept if >90% of the 970 annotated features are detected (target 90.1% ID rate) and ~81.7% are accurately quantified.

## Related tools

- **UmetaFlow** (End-to-end workflow that orchestrates file conversion, feature detection, alignment, adduct annotation, re-quantification and blank/minfrac filtering) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS 3.0** (Underlying tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking invoked via the Snakemake rules) — https://github.com/biosustain/snakemake_UmetaFlow
- **Snakemake** (Workflow engine running the pre-processing and re-quantification steps with per-rule conda environments) — https://github.com/biosustain/snakemake_UmetaFlow
- **pyOpenMS** (Python bindings to the OpenMS algorithms used in the Jupyter-notebook implementation of the same pre-processing pipeline) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **ThermoRawFileParser** (Converts Thermo RAW files into centroided mzML in the file conversion step (referenced in the MacOS setup script))

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- On MTBLS736, >90% of the 970 annotated ground-truth features should be detected (reported 90.1% true feature ID rate).
- About 81.7% of detected features should be accurately quantified against the MTBLS736 benchmark.
- Blank/QC removal and minfrac filtering should leave a feature table in which consensus features with too many missing values have been removed — verify no feature exceeds the missing-value threshold in samples.tsv samples.
- Re-quantification output must contain an entry for every detected feature in every sample file, eliminating missing values for statistical analysis.
- The snakemake --dry-run must resolve all OpenMS tool calls (feature detection, map alignment, adduct annotation, feature linking, re-quantification) without missing input dependencies.

## Limitations

- Performance numbers (90.1% ID rate, 81.7% quantification rate) are benchmark-specific to MTBLS733/MTBLS736 and may not transfer to other instrument types or study designs.
- File conversion is only automated for Thermo RAW files; Agilent/Bruker data must be converted independently with ProteoWizard before entering the workflow.
- Some benchmark table details (e.g. Table S5 feature identification, quantification and marker selection numbers) are only in supplementary files, not the main text.
- No explicit software version numbers were stated for UmetaFlow, OpenMS, or Snakemake in the article text.

## Evidence

- [intro] OpenMS 3.0 tools perform feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking"
- [readme] This step includes an optional removal of blank/QC samples if defined by the user. Optional minfrac step here allows for removal of consensus features with too many missing values.: "This step includes an optional removal of blank/QC samples if defined by the user. Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] Re-quantify all raw files to avoid missing values resulted by the pre-processing workflow for statistical analysis and data exploration (optional step).: [evidence span withheld — anchor status: external_doc]
- [abstract] UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection.: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
- [discussion] Benchmarked it using the publicly available datasets MTBLS733 and MTBLS736.: "benchmarked it using the publicly available datasets MTBLS733 and MTBLS736"
