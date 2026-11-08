---
name: cross-tool-feature-count-comparison
description: 'Decision support for selecting an LC/MS signal-detection/peak-picking
  tool: you have raw mzML files, candidate peak-detection algorithms (e.g. mzLearn,
  XCMS, ASARI), and need objective per-dataset feature counts plus TP/FP rates to
  compare them.'
when_to_use_negative:
- When the goal is downstream annotation or biological interpretation of already-detected
  features (e.g. de-isotoping, MS/MS identification) — the article states mzLearn
  results are at the ion MS1 level and it does not perform de-isotoping or MS/MS identification
- When the inputs are already normalized feature intensity matrices rather than raw
  mzML files — this skill compares tool behavior on raw signal-detection, not downstream
  modeling on existing tables
- When datasets are so small that feature counts alone cannot distinguish true signal
  from noise — the article's benchmarks span 15 public datasets and 20,548 blood samples,
  and per-dataset TP/FP estimates require enough peaks to be stable
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzLearn
  role: Candidate signal-detection tool that runs from mzML without user-set parameters;
    benchmarked for signal count and TP/FP rate
- name: XCMS
  role: Baseline peak-detection and alignment tool run on the same datasets for feature-count
    and TP/FP comparison
- name: ASARI
  role: Baseline peak-detection tool run on the same datasets for feature-count and
    TP/FP comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:19c3afee455ca7d658a06774d1ba029ae6a13e508c133342de7ecde6f387c62a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - ASARI
  - XCMS
  - mzLearn
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-tool-feature-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill that tabulates the number of MS1 signals detected by different LC/MS peak-detection tools (mzLearn, XCMS, ASARI) on the same raw mzML datasets and compares their true-positive and false-positive rates against peak-shape ground truth, to justify tool choice for feature-detection pipelines.

## When to use

Decision support for selecting an LC/MS signal-detection/peak-picking tool: you have raw mzML files, candidate peak-detection algorithms (e.g. mzLearn, XCMS, ASARI), and need objective per-dataset feature counts plus TP/FP rates to compare them.

## When NOT to use

- When the goal is downstream annotation or biological interpretation of already-detected features (e.g. de-isotoping, MS/MS identification) — the article states mzLearn results are at the ion MS1 level and it does not perform de-isotoping or MS/MS identification
- When the inputs are already normalized feature intensity matrices rather than raw mzML files — this skill compares tool behavior on raw signal-detection, not downstream modeling on existing tables
- When datasets are so small that feature counts alone cannot distinguish true signal from noise — the article's benchmarks span 15 public datasets and 20,548 blood samples, and per-dataset TP/FP estimates require enough peaks to be stable

## Inputs

- Raw LC/MS dataset files in mzML format (e.g. 15 public targeted-metabolomics benchmark datasets)
- Peak-shape-based ground-truth labels for true/false feature calls (e.g. manual peak-shape annotation of the benchmark datasets)
- Output of each candidate peak-detection tool run on the same mzML inputs (mzLearn, XCMS, ASARI feature tables)

## Outputs

- Per-dataset, per-tool table of detected MS1 signal counts (with per-tool averages, e.g. 11,442 / 7,100 / 4,655)
- Per-tool true-positive and false-positive rate estimates against peak-shape ground truth (e.g. TP 89.0% / 77.4% / 49.6%; FP 12.5% / 17.3% / 18.8%)
- Combined comparison table such as benchmark_results.csv

## How to apply

Retrieve the benchmark datasets as mzML files (e.g. the 15 public targeted-metabolomics studies enumerated in the article's Supplementary Table 1). Run each candidate tool on every dataset: mzLearn runs directly from mzML with no user-set parameters, while XCMS and ASARI are run through their standard peak-detection and alignment workflows with their respective default parameter sets. Tabulate the number of detected MS1 signals per dataset per tool and compute per-tool averages — the article's reference values are 11,442 signals for mzLearn, 7,100 for XCMS, and 4,655 for ASARI. Assess detected features against peak-shape-based ground-truth labels to compute true-positive rates (89.0%, 77.4%, 49.6% respectively) and false-positive rates (12.5%, 17.3%, 18.8%). Compile counts, TP rates, and FP rates into a single comparison table (e.g. benchmark_results.csv); the decision point is that a tool that maximizes signal count must also be checked for low FP rate, since more detected peaks can simply reflect more noise.

## Related tools

- **mzLearn** (Candidate signal-detection tool that runs from mzML without user-set parameters; benchmarked for signal count and TP/FP rate)
- **XCMS** (Baseline peak-detection and alignment tool run on the same datasets for feature-count and TP/FP comparison)
- **ASARI** (Baseline peak-detection tool run on the same datasets for feature-count and TP/FP comparison)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Per-tool average signal counts reproduce the published reference values: mzLearn ≈ 11,442, XCMS ≈ 7,100, ASARI ≈ 4,655 across 15 datasets
- True-positive rates match the published figures (mzLearn 89.0%, XCMS 77.4%, ASARI 49.6%) when evaluated against peak-shape ground truth
- False-positive rates are consistent with the reported numbers (mzLearn 12.5%, XCMS 17.3%, ASARI 18.8%)
- The comparison table contains a row per dataset per tool with no missing entries, and the ranking by TP rate (mzLearn > XCMS > ASARI) is preserved across datasets
- A tool that reports more total signals than mzLearn but a higher FP rate than mzLearn's 12.5% is not an automatic winner — counts and FP rate must be reported together

## Limitations

- TP/FP evaluation depends on peak-shape ground-truth labels, which require expert review and manual labeling and may not transfer to new datasets
- Computational cost is high: iterative learning in mzLearn can take up to 2.5 days on a single machine, and parallel processing on multiple compute nodes is not yet implemented
- The benchmark relies on the specific 15 public datasets listed in the article; results may not generalize to other LC modes or experimental designs
- The article's benchmark covers LC/MS MS1 signal detection; tools differing in MS/MS handling or chromatographic alignment beyond peak picking may need additional evaluation

## Evidence

- [abstract] mzLearn detects 11,442 signals on average vs 7,100 (XCMS) and 4,655 (ASARI): "mzLearn detects 11,442 signals on average vs 7,100 (XCMS) and 4,655 (ASARI)"
- [abstract] with higher TP (89.0% vs 77.4% vs 49.6%) and lower FP (12.5% vs 17.3% vs 18.8%): "with higher TP (89.0% vs 77.4% vs 49.6%) and lower FP (12.5% vs 17.3% vs 18.8%)"
- [abstract] a data-driven MS1 signal-detection and alignment method that runs from mzML files without user-set parameters: "a data-driven MS1 signal-detection and alignment method that runs from mzML files without user-set parameters"
- [discussion] it iteratively refines signal characteristics and filters out the noise to identify the missing true peaks: "it iteratively refines signal characteristics and filters out the noise to identify the missing true peaks"
