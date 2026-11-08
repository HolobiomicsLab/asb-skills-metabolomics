---
name: ground-truth-feature-matching
description: Use when an untargeted metabolomics feature table (e.g. from UmetaFlow/OpenMS
  on Thermo Q Exactive mzML data) must be validated against a known list of targeted
  ground-truth features to report detection, quantification and marker-selection performance.
when_to_use_negative:
- No targeted ground-truth feature list exists for the dataset — there is nothing
  to match against and rates cannot be computed
- Input is raw vendor files (Thermo raw) rather than a processed feature quantification
  matrix — pre-processing/re-quantification must be run first
- The goal is annotation (SIRIUS/CSI:FingerID, GNPS FBMN/IIMN) rather than benchmarking
  detection, quantification or marker selection
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: UmetaFlow
  role: Produces the untargeted feature quantification matrix that is benchmarked
    against the targeted ground truth
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS
  role: Provides feature detection, map alignment, feature grouping and re-quantification
    algorithms whose output is matched to ground truth
- name: Snakemake
  role: Runs the workflow whose config.yaml flags (TRUE/FALSE step toggles, positive/negative
    ionisation) and minfrac filters parameterize the benchmarked output
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Python
  role: Implements the ground-truth matching and metric computation (detection rate,
    accurate quantification rate, marker counts)
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:4127922e7be34e281060776f412077c30ddd92de6b8fa8e4f271f521b183236c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - Jupyter
  - OpenMS
  - Python
  - Snakemake
  - UmetaFlow
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# ground-truth-feature-matching

## Summary

Benchmarks untargeted LC-MS/MS feature detection, quantification and marker selection output against a targeted-analysis ground truth (e.g. the 836 ground-truth features of MTBLS733), computing identification, accurate-quantification and marker precision/recall rates.

## When to use

Use when an untargeted metabolomics feature table (e.g. from UmetaFlow/OpenMS on Thermo Q Exactive mzML data) must be validated against a known list of targeted ground-truth features to report detection, quantification and marker-selection performance.

## When NOT to use

- No targeted ground-truth feature list exists for the dataset — there is nothing to match against and rates cannot be computed
- Input is raw vendor files (Thermo raw) rather than a processed feature quantification matrix — pre-processing/re-quantification must be run first
- The goal is annotation (SIRIUS/CSI:FingerID, GNPS FBMN/IIMN) rather than benchmarking detection, quantification or marker selection

## Inputs

- Re-quantified untargeted feature quantification matrix (TSV) from UmetaFlow/OpenMS pre-processing and re-quantification steps
- Targeted-analysis ground-truth feature list (836 features for MTBLS733)
- Group/condition labels for samples (SA/SB Piper nigrum standard mixtures, 5 replicates per group)
- Discriminating marker list from targeted analysis for marker-selection benchmarking

## Outputs

- Benchmark metrics table (e.g. MTBLS733_feature_quantification_benchmark.csv) with detection rate (93.1%), accurate quantification rate (94.6%), and marker selection counts (47 true / 5 false of 50)
- Matched feature table linking untargeted consensus features to ground-truth features

## How to apply

Run the untargeted pipeline (UmetaFlow pre-processing with OpenMS feature detection, map alignment and feature grouping, plus the optional re-quantification step and minfrac filter) to produce the final feature quantification matrix, then match its consensus features against the targeted ground-truth list — for MTBLS733 this is 836 targeted features from the SA/SB Piper nigrum standard mixtures (5 replicates per group). Count how many ground-truth features the untargeted workflow detected (778/836 = 93.1% after parameter optimization) and how many of those were accurately quantified (736/836 = 94.6%). For marker selection, evaluate discriminating markers between the SA and SB groups against the targeted marker list (reported: 47 true and 5 false markers out of 50). Write the re-quantified feature table and the computed benchmark metrics (e.g. MTBLS733_feature_quantification_benchmark.csv) so results can be compared against other tools such as XCMS.

## Related tools

- **UmetaFlow** (Produces the untargeted feature quantification matrix that is benchmarked against the targeted ground truth) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS** (Provides feature detection, map alignment, feature grouping and re-quantification algorithms whose output is matched to ground truth)
- **Snakemake** (Runs the workflow whose config.yaml flags (TRUE/FALSE step toggles, positive/negative ionisation) and minfrac filters parameterize the benchmarked output) — https://github.com/biosustain/snakemake_UmetaFlow
- **Python** (Implements the ground-truth matching and metric computation (detection rate, accurate quantification rate, marker counts)) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all  # run UmetaFlow pre-processing + re-quantification, then compare the re-quantified feature table to the 836 MTBLS733 targeted features and write MTBLS733_feature_quantification_benchmark.csv
```

## Evaluation signals

- Detection rate is computed as detected true features / 836 targeted ground-truth features; reported reference value is 778/836 = 93.1% after parameter optimization
- Accurate quantification rate is computed over detected features; reported reference value is 736/836 = 94.6%
- Marker selection evaluation should reproduce 47 true and 5 false markers out of 50 reported markers on MTBLS733
- The final benchmark output is a CSV (MTBLS733_feature_quantification_benchmark.csv) containing the re-quantified feature table plus the computed metrics
- Detection rate should exceed 90%, consistent with the paper's claim that UmetaFlow detected more than 90% of all ground truth features on MTBLS733 and MTBLS736

## Limitations

- Benchmark performance numbers (Tables S5, S7) are only in supplementary files, not in the main text
- The MTBLS733 result depends on parameter optimization and the optional re-quantification and minfrac steps; skipping them changes the detection/quantification rates
- Only 836 targeted ground-truth features for MTBLS733 define the reference — features absent from the targeted list cannot be scored

## Evidence

- [results] On MTBLS733, after parameter optimization, UmetaFlow detected 778 true features (a 93.1% untargeted versus targeted identification rate), and 736 of those were accurately quantified (94.6%).: [evidence span withheld — anchor status: not_found]
- [abstract] On MTBLS733 and MTBLS736, UmetaFlow detected more than 90% of all ground truth features: "UmetaFlow detected more than 90% of all ground truth features and performed exceptionally well in quantification and discriminating marker selection"
- [readme] Re-quantification step to recover all features with missing values, minfrac filter removing consensus features with too many missing values: "Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration. Optional "minfrac" step here allows for"
- [methods] Benchmark the output against the 836 targeted ground-truth features: compute the detection rate and accurate quantification rate: [evidence span withheld — anchor status: not_found]
