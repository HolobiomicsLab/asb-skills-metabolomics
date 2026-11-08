---
name: tic-normalization-benchmarking
description: Use when you have LC/MS MS1 ion-feature matrices (e.g., mzLearn mzML-derived
  outputs, >150 samples per dataset) and need to quantify whether drift correction
  is necessary by comparing synthetic QC vs experimental QC vs TIC vs raw normalization
  via RSD on withheld QC injections.
when_to_use_negative:
- The feature matrix is already normalized or you are doing downstream pretraining/fine-tuning
  (e.g., VAE embedding) rather than judging which normalization to use
- The dataset contains no identifiable QC injections, so RSD across hidden QC samples
  cannot be computed (public metabolomics repositories often lack standardized QC
  samples)
- The intensities come from direct-infusion or MS/MS-level measurements where mzLearn's
  run-order/intensity drift model does not apply
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_2269
tools:
- name: mzLearn
  role: Runs data-driven drift correction with internally generated synthetic QC samples
    on the ion-feature matrices; produces the MS1 ion-feature tables whose hidden-QC
    RSD is benchmarked
  repo: github.com/ReviveMed/mzEmbed
- name: mzEmbed
  role: Codebase whose mzLearn outputs feed the normalization-evaluation workflow;
    QC RSD comparison is computed on mzLearn-derived feature matrices
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:a97667307dc7a7979ed48a756d4049106d828f74cc3259442a81742a98d130ea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - mzEmbed
  - mzLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# tic-normalization-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark LC/MS feature-intensity normalization strategies — mzLearn's synthetic QC drift correction, withheld experimental QC correction, total-ion-current (TIC) normalization, and no normalization (raw) — by comparing relative standard deviation (RSD = CV × 100) across hidden QC injections.

## When to use

Use when you have LC/MS MS1 ion-feature matrices (e.g., mzLearn mzML-derived outputs, >150 samples per dataset) and need to quantify whether drift correction is necessary by comparing synthetic QC vs experimental QC vs TIC vs raw normalization via RSD on withheld QC injections.

## When NOT to use

- The feature matrix is already normalized or you are doing downstream pretraining/fine-tuning (e.g., VAE embedding) rather than judging which normalization to use
- The dataset contains no identifiable QC injections, so RSD across hidden QC samples cannot be computed (public metabolomics repositories often lack standardized QC samples)
- The intensities come from direct-infusion or MS/MS-level measurements where mzLearn's run-order/intensity drift model does not apply

## Inputs

- mzLearn MS1 ion-feature matrices (from mzML files) for normalization-evaluation datasets, each with >150 samples
- Sample-role annotations identifying experimental QC injections to withhold (hide) from normalization
- Dataset partition assignments for per-dataset aggregation

## Outputs

- qc_rsd_results.csv tabulating per-dataset and aggregated RSD (%) for the four conditions: synthetic QC, experimental QC, TIC, raw
- Per-dataset RSD values (coefficient of variation × 100) computed on hidden QC injections

## How to apply

Load the mzLearn ion-feature matrices for each normalization-evaluation dataset (>150 samples), keeping the experimental QC injections hidden (withheld) from all normalization processing. Run mzLearn's data-driven drift correction, which internally generates synthetic QC samples and constructs intensity and rt drift maps through dynamic, iterative learning. Apply the three comparator treatments on the identical feature matrices: correction using the withheld experimental QC samples, per-sample TIC normalization, and raw (no-normalization) intensities. For each treatment and dataset, compute the RSD (coefficient of variation × 100) of feature intensities across the hidden QC injections, then aggregate per-dataset RSDs into a side-by-side four-condition table written to qc_rsd_results.csv. Validate by checking the aggregate means against the reported benchmarks of 17% (synthetic QC), 16% (experimental QC), 23% (TIC), and 25% (raw) as shown in Fig. 3c / Supplementary Table 4; synthetic QC should land within ~1 percentage point of experimental QC, and TIC/raw should be clearly worse.

## Related tools

- **mzLearn** (Runs data-driven drift correction with internally generated synthetic QC samples on the ion-feature matrices; produces the MS1 ion-feature tables whose hidden-QC RSD is benchmarked) — github.com/ReviveMed/mzEmbed
- **mzEmbed** (Codebase whose mzLearn outputs feed the normalization-evaluation workflow; QC RSD comparison is computed on mzLearn-derived feature matrices) — github.com/ReviveMed/mzEmbed

## Examples

```
import pandas as pd; (qc_df.std()/qc_df.mean()*100).to_csv("qc_rsd_results.csv")  # RSD = CV x 100 across hidden QC injections
```

## Evaluation signals

- Aggregated mean RSDs match the reported benchmarks: ~17% (synthetic QC), ~16% (experimental QC), ~23% (TIC), ~25% (raw), as in Fig. 3c / Supplementary Table 4
- Synthetic QC normalization's RSD in hidden QC samples is comparable to (within ~1 percentage point of) experimental QC normalization on every normalization-evaluation dataset
- TIC and raw baselines show strictly higher mean RSD than both QC-based corrections across the aggregated datasets
- RSD is computed only across withheld (hidden) experimental QC injections per dataset, never using them inside the normalization itself for the synthetic-QC arm
- qc_rsd_results.csv contains all four conditions side by side for each of the six normalization-evaluation datasets

## Limitations

- Public metabolomics repositories often lack standardized QC samples, so this hidden-QC RSD benchmark cannot be reproduced on every dataset
- mzLearn's iterative drift learning is computationally expensive — analysis can take up to 2.5 days on a single machine with no multi-node parallelization implemented
- mzLearn results are at the ion MS1 level and do not include de-isotoping or MS/MS identification, so the benchmark covers only MS1 feature intensities

## Evidence

- [results] synthetic QC normalization achieved 17% RSD in hidden QC samples, comparable to 16% RSD when normalizing with experimental QC samples: [evidence span withheld — anchor status: not_found]
- [readme] Iterative learning: mzLearn autonomously refines signal detection, correcting for retention time (rt) and intensity drifts caused by batch effects and run order.: [evidence span withheld — anchor status: not_found]
