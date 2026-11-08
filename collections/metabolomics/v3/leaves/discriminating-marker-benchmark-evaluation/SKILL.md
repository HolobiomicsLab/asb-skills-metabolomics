---
name: discriminating-marker-benchmark-evaluation
description: Use when an untargeted LC-MS/MS feature quantification table must be
  scored for marker-selection performance against a curated ground-truth feature list
  (e.g. 836 targeted MTBLS733 features).
when_to_use_negative:
- The dataset has no targeted ground-truth feature list, so detection/quantification/marker
  rates cannot be computed against known features
- The goal is annotation or structural elucidation (SIRIUS/CSI:FingerID, GNPS FBMN/IIMN)
  rather than benchmarking feature detection, quantification and marker selection
- Input is already a scored benchmark table or the software's published performance
  numbers are sufficient for the question
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: UmetaFlow
  role: Untargeted metabolomics workflow (Snakemake and pyOpenMS implementations)
    whose feature detection, re-quantification and discriminating marker selection
    output is benchmarked
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: OpenMS
  role: Provides the underlying algorithms (feature detection, map alignment, re-quantification,
    feature linking) via pyOpenMS bindings
- name: Snakemake
  role: Workflow engine executing the configurable pipeline whose parameters are optimized
    before benchmarking
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Python
  role: Language used (with pyOpenMS in Jupyter notebooks) to compute benchmark metrics
    and write the feature quantification benchmark table
  repo: https://github.com/biosustain/pyOpenMS_UmetaFlow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:7d8d1497f86826ffc5a8b4f281a36f859176364b92141f17be58701fee30180d
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# discriminating-marker-benchmark-evaluation

## Summary

Benchmarking skill for evaluating how well an untargeted metabolomics workflow (e.g. UmetaFlow on the MTBLS733 Piper nigrum SA/SB mixtures) selects true discriminating markers against a targeted ground-truth feature list.

## When to use

Use when an untargeted LC-MS/MS feature quantification table must be scored for marker-selection performance against a curated ground-truth feature list (e.g. 836 targeted MTBLS733 features).

## When NOT to use

- The dataset has no targeted ground-truth feature list, so detection/quantification/marker rates cannot be computed against known features
- The goal is annotation or structural elucidation (SIRIUS/CSI:FingerID, GNPS FBMN/IIMN) rather than benchmarking feature detection, quantification and marker selection
- Input is already a scored benchmark table or the software's published performance numbers are sufficient for the question

## Inputs

- Re-quantified feature quantification matrix (TSV) from UmetaFlow pre-processing and re-quantification steps
- Targeted ground-truth feature list (836 features for MTBLS733)
- config/dataset.tsv, config/samples.tsv and config/blanks.tsv listing SA/SB Piper nigrum samples (5 replicates per group) and blanks
- Centroided mzML files in data/mzML/ (converted from Thermo Q Exactive raw files)

## Outputs

- Benchmark metrics CSV (e.g. MTBLS733_feature_quantification_benchmark.csv) with detection rate, accurate quantification rate, and true/false marker counts
- Marker selection performance: number of true and false discriminating markers within the selected marker set (reported: 47 true, 5 false out of 50)

## How to apply

First produce the re-quantified feature matrix by running the UmetaFlow pipeline (Snakemake or pyOpenMS) on the dataset: convert Thermo raw files to centroided mzML via ThermoRawFileParser, then run OpenMS 3.0 feature detection, map alignment, feature grouping, and optional re-quantification with the optional minfrac filter to remove consensus features with too many missing values. For MTBLS733 (Thermo Q Exactive, SA/SB Piper nigrum standard mixtures, 5 replicates per group), compare the untargeted output against the 836 targeted ground-truth features and compute the detection rate (reported: 778 true features, 93.1%) and accurate quantification rate (reported: 736 features, 94.6%). For discriminating marker selection specifically, compare the set of selected discriminating markers (e.g. top 50) against the ground truth and count true versus false markers; the published benchmark reports 47 true and 5 false markers out of 50 on MTBLS733. Write the re-quantified feature table and the computed metrics to a CSV such as MTBLS733_feature_quantification_benchmark.csv; note that detailed comparison tables (e.g. Table S5) live in the supplementary files.

## Related tools

- **UmetaFlow** (Untargeted metabolomics workflow (Snakemake and pyOpenMS implementations) whose feature detection, re-quantification and discriminating marker selection output is benchmarked) — https://github.com/biosustain/snakemake_UmetaFlow
- **OpenMS** (Provides the underlying algorithms (feature detection, map alignment, re-quantification, feature linking) via pyOpenMS bindings)
- **Snakemake** (Workflow engine executing the configurable pipeline whose parameters are optimized before benchmarking) — https://github.com/biosustain/snakemake_UmetaFlow
- **Python** (Language used (with pyOpenMS in Jupyter notebooks) to compute benchmark metrics and write the feature quantification benchmark table) — https://github.com/biosustain/pyOpenMS_UmetaFlow

## Examples

```
snakemake --use-conda --cores all
```

## Evaluation signals

- Detection rate on MTBLS733 should approach the reported 778/836 true features (93.1% untargeted versus targeted identification rate) after parameter optimization
- Accurate quantification rate should approach the reported 736 accurately quantified features (94.6%)
- Discriminating marker selection should yield roughly 47 true and 5 false markers out of 50 selected, as reported for MTBLS733
- The final benchmark output (MTBLS733_feature_quantification_benchmark.csv) contains the re-quantified feature table plus the computed detection, quantification and marker metrics
- More than 90% of ground-truth features detected, consistent with the paper's abstract claim for MTBLS733 and MTBLS736

## Limitations

- Benchmark performance numbers (e.g. Tables S5 and S7 comparing software on MTBLS733) are only in supplementary files, not the main text
- No software version numbers are stated for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake, complicating exact reproduction
- No explicit limitations or caveats section is stated in the provided article text

## Evidence

- [abstract] On MTBLS733 and MTBLS736, UmetaFlow detected more than 90% of all ground truth features: [evidence span withheld — anchor status: not_found]
- [methods] Optional minfrac filter step: "Optional "minfrac" step here allows for removal of consensus features with too many missing values."
- [discussion] Benchmark table only in supplementary material: "Table S5. Feature identification, quantification, and marker selection performance of different untargeted metabolomic data processing software using the benchmark dataset MTBLS733"
- [readme] Pipeline steps and file conversion: [evidence span withheld — anchor status: external_doc]
- [readme] Re-quantification rationale: "Re-quantify all features with missing values across samples resulted from the pre-processing step for more reliable statistical analysis and data exploration."
