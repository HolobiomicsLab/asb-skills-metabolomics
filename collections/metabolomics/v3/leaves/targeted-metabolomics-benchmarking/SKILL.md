---
name: targeted-metabolomics-benchmarking
description: Use when raw LC/MS mzML files from targeted metabolomics runs must be
  processed by multiple peak-detection tools and their signal counts and TP/FP rates
  compared to choose a detector.
when_to_use_negative:
- Input is already a curated feature table (the mzLearn/mzEmbed CSV format) rather
  than raw mzML
- Only one peak-detection tool is being used with no intent to compare against alternatives
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzLearn
  role: Primary signal-detection tool; runs directly from mzML files without user-set
    parameters
  repo: https://github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Baseline peak-detection and alignment tool for comparison
- name: ASARI
  role: Baseline signal-detection tool for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:fb2e93da09d1e3d1102cc2da7c66bc442c2446abf928e09aece8713f10b6709f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  repo_url: https://github.com/ReviveMed/mzEmbed
  tools:
  - ASARI
  - XCMS
  - mzLearn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# targeted-metabolomics-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks MS1 signal-detection tools (mzLearn, XCMS, ASARI) on 15 public targeted-metabolomics mzML datasets by comparing average detected signals, true-positive rates, and false-positive rates against peak-shape ground-truth labels.

## When to use

Use when raw LC/MS mzML files from targeted metabolomics runs must be processed by multiple peak-detection tools and their signal counts and TP/FP rates compared to choose a detector.

## When NOT to use

- Input is already a curated feature table (the mzLearn/mzEmbed CSV format) rather than raw mzML
- Only one peak-detection tool is being used with no intent to compare against alternatives

## Inputs

- 15 public targeted-metabolomics mzML datasets (Supplementary Table 1)

## Outputs

- Per-tool average MS1 signal counts across datasets
- Per-tool true-positive and false-positive rates vs. peak-shape ground truth
- Comparison table (e.g. benchmark_results.csv)

## How to apply

Retrieve the 15 public targeted-metabolomics benchmark datasets enumerated in Supplementary Table 1 as mzML files. Process each dataset identically with mzLearn (which runs from mzML without user-set parameters), XCMS, and ASARI using their respective peak-detection and alignment workflows. Tabulate detected MS1 signals per dataset per tool and compute averages (mzLearn 11,442; XCMS 7,100; ASARI 4,655). Assess detected features against peak-shape-based ground-truth labels to compute TP rates (89.0%/77.4%/49.6%) and FP rates (12.5%/17.3%/18.8%). Rationale: mzLearn's zero-parameter, iterative data-driven learning refines signal characteristics and filters noise, identifying missing true peaks that parameter-tuned tools miss.

## Related tools

- **mzLearn** (Primary signal-detection tool; runs directly from mzML files without user-set parameters) — https://github.com/ReviveMed/mzEmbed
- **XCMS** (Baseline peak-detection and alignment tool for comparison)
- **ASARI** (Baseline signal-detection tool for comparison)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Average detected signals reproduce 11,442 (mzLearn), 7,100 (XCMS), 4,655 (ASARI)
- True-positive rates match 89.0%, 77.4%, 49.6% respectively
- False-positive rates match 12.5%, 17.3%, 18.8% respectively
- mzLearn detects >60% more signals on average than existing methods
- False-positive rates submitted for expert review against manual peak-shape labeling

## Limitations

- mzLearn is deterministic and iterative; large cohorts can take up to 2.5 days on a single machine and parallel processing across compute nodes is not yet implemented
- mzLearn outputs ion-level MS1 features only — no de-isotoping, MS/MS identification, or compound annotation
- mzLearn's pre-trained model is specific to HILIC positive mode; other LC modes are not covered

## Evidence

- [readme] README zero-parameter design: [evidence span withheld — anchor status: not_found]
