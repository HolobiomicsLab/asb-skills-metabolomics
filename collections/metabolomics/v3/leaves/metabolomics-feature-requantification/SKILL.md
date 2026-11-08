---
name: metabolomics-feature-requantification
description: Use when an untargeted LC-MS/MS feature table from OpenMS pre-processing
  contains missing values across samples and a complete quantification matrix is needed
  for statistical analysis, marker selection, or benchmarking detection/quantification
  rates.
when_to_use_negative:
- Input is already a complete feature quantification table with no missing values
  (e.g. output of a prior re-quantification run)
- The data are not untargeted LC-MS/MS metabolomics profiles (e.g. targeted assays
  or non-OpenMS feature tables)
- The goal is annotation or identification (SIRIUS/CSI:FingerID, GNPS FBMN/IIMN, spectral
  matching) rather than filling quantification gaps
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: UmetaFlow
  role: Workflow (Snakemake and pyOpenMS/Jupyter implementations) whose step 3 performs
    re-quantification of features with missing values
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: OpenMS
  role: Provides the underlying re-quantification (and pre-processing) algorithms
    via OpenMS 3.0 tools and pyOpenMS bindings
- name: Snakemake
  role: Orchestrates the re-quantification rule, controlled by TRUE/FALSE step flags
    in config.yaml
  repo: https://github.com/biosustain/snakemake_UmetaFlow.git
- name: pyOpenMS
  role: Python bindings to OpenMS algorithms used in the Jupyter notebook implementation
    of the re-quantification step
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
- name: ThermoRawFileParser
  role: Converts Thermo raw files (e.g. Thermo Q Exactive data from MTBLS733) to centroided
    mzML upstream of re-quantification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:11c9b473255815fc640dbb0c61108aab400164ba6b7e1c210ffde4449571b94b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow.git
  tools:
  - Jupyter
  - OpenMS
  - Python
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolomics-feature-requantification

## Summary

UmetaFlow's OpenMS-based re-quantification step recovers intensities for features with missing values across LC-MS/MS samples after pre-processing (feature detection, alignment, grouping), producing a complete feature quantification matrix. It was benchmarked on MTBLS733 (Thermo Q Exactive, SA/SB Piper nigrum mixtures) where 736 of 778 detected features were accurately quantified (94.6%).

## When to use

Use when an untargeted LC-MS/MS feature table from OpenMS pre-processing contains missing values across samples and a complete quantification matrix is needed for statistical analysis, marker selection, or benchmarking detection/quantification rates.

## When NOT to use

- Input is already a complete feature quantification table with no missing values (e.g. output of a prior re-quantification run)
- The data are not untargeted LC-MS/MS metabolomics profiles (e.g. targeted assays or non-OpenMS feature tables)
- The goal is annotation or identification (SIRIUS/CSI:FingerID, GNPS FBMN/IIMN, spectral matching) rather than filling quantification gaps

## Inputs

- Centroided mzML files in data/mzML/ (Thermo raw files converted via ThermoRawFileParser, or Agilent/Bruker files converted externally with proteowizard)
- Pre-processed consensus feature table (from OpenMS feature detection, map alignment and feature grouping) containing features with missing values across samples
- config/dataset.tsv, config/samples.tsv and config/blanks.tsv listing sample filenames and blank/QC designations

## Outputs

- Re-quantified feature quantification matrix (TSV under results/), with missing values filled across samples
- Optionally minfrac-filtered feature table (consensus features with too many missing values removed)
- Benchmark metrics for MTBLS733-style evaluations (e.g. accurate quantification rate: 736/778 features, 94.6%)

## How to apply

After the UmetaFlow pre-processing step (OpenMS 3.0 feature detection, map alignment, feature grouping, with optional blank/QC sample removal and minfrac filtering) has produced a consensus feature table with missing values, run the optional re-quantification step to re-quantify all features with missing values across samples, recovering intensities the pre-processing pipeline missed. An optional second minfrac filter can then remove consensus features that still have too many missing values, yielding the final feature quantification matrix. In the Snakemake implementation, toggle this step via TRUE/FALSE flags in config.yaml; in the pyOpenMS implementation it is notebook 3_Requantification(optional).ipynb. The rationale is that pre-processing (detection/alignment/grouping) leaves gaps per sample, and re-quantification gives more reliable downstream statistics — on MTBLS733 it enabled 736 of 778 true features (94.6% of detected) to be accurately quantified against 836 targeted ground-truth features.

## Related tools

- **UmetaFlow** (Workflow (Snakemake and pyOpenMS/Jupyter implementations) whose step 3 performs re-quantification of features with missing values) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **OpenMS** (Provides the underlying re-quantification (and pre-processing) algorithms via OpenMS 3.0 tools and pyOpenMS bindings)
- **Snakemake** (Orchestrates the re-quantification rule, controlled by TRUE/FALSE step flags in config.yaml) — https://github.com/biosustain/snakemake_UmetaFlow.git
- **pyOpenMS** (Python bindings to OpenMS algorithms used in the Jupyter notebook implementation of the re-quantification step) — https://github.com/biosustain/pyOpenMS_UmetaFlow
- **ThermoRawFileParser** (Converts Thermo raw files (e.g. Thermo Q Exactive data from MTBLS733) to centroided mzML upstream of re-quantification)

## Examples

```
snakemake --use-conda --cores all  # with re-quantification step set to TRUE in config.yaml (or run 3_Requantification(optional).ipynb in pyOpenMS_UmetaFlow)
```

## Evaluation signals

- All features that had missing values after pre-processing now have intensities across samples in the final feature matrix
- On MTBLS733, benchmark against 836 targeted ground-truth features: expect ~778 true detections (93.1%) and ~736 accurately quantified (94.6%) after parameter optimization
- Optional minfrac filter has removed consensus features with too many missing values from the output matrix
- Snakemake dry-run (snakemake --use-conda --dry-run) resolves the re-quantification rule without errors given the config.yaml flags and TSV files
- Output feature table is in results/ in TSV format readable as a Pandas dataframe

## Limitations

- Re-quantification is an optional step; skipping it leaves missing values from pre-processing that degrade statistical analysis
- Even with re-quantification, not all features are recovered or accurately quantified (on MTBLS733, 778 of 836 ground-truth features detected; 736 of those accurately quantified)
- Benchmark performance numbers are only in supplementary files (Tables S5, S7), not the main text, and software version numbers for UmetaFlow/OpenMS are not stated in the article
- The Snakemake implementation targets Linux and MacOS only

## Evidence

- [readme] Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration.: "Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration. Optional "minfrac" step"
- [intro] OpenMS 3.0 tools for re-quantification: "using OpenMS [17] 3.0 tools for feature detection, map alignment, adduct annotation, re-quantification and feature linking, spectral matching"
- [abstract] benchmarking detection/quantification rates: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
- [methods] minfrac filter: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [readme] optional step in both implementations: [evidence span withheld — anchor status: external_doc]
