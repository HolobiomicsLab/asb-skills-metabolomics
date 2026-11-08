---
name: cross-dataset-peak-recovery-assessment
description: Use when replicating or validating an MSI peak-identification pipeline
  against a published peak list (e.g., the 63 peaks of Zhang et al. 2023) using multiple
  replicate DESI-MSI datasets.
when_to_use_negative:
- The input is raw, unprocessed imaging data with no per-replicate workspace files
  available for pseudobulk spectrum construction.
- There is no published or curated reference peak list to compare against, so recovery
  cannot be quantified.
- The goal is de novo metabolite annotation (e.g., METASPACE or Metlin/LOTUS database
  searching) rather than benchmarking peak-extraction reproducibility.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3311
tools:
- name: DIMPLE
  role: Pipeline whose consensus-peak extraction (2.5 ppm tolerance, 1.5-fold background
    suppression, sparsity filtering) is being benchmarked for cross-replicate peak
    recovery
  repo: github.com/dickinsonlab/DIMPLE-code
- name: DESI-MSI
  role: Source modality supplying the per-pixel mass spectra of the three B73 maize
    root replicates used for recovery assessment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.09.22.677919
    title: Computational method for mapping mass signatures along developmental gradients
      reveals a novel role for a monosaccharide tetrose in maize salt-stress response
  source_content_hash: sha256:be9e540c418eba2d193d1a6d1a96ccbea82c98d58848915602debc4bffde14fd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/dickinsonlab/DIMPLE-code
  tools:
  - DESI-MSI
  - DIMPLE
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3311
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-dataset-peak-recovery-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a peak-extraction pipeline (here DIMPLE applied to DESI-MSI data) by recovering previously published mass signatures across replicate datasets, quantifying per-dataset counts, pooled unique signatures, and percent recovery against a reference peak list.

## When to use

Use when replicating or validating an MSI peak-identification pipeline against a published peak list (e.g., the 63 peaks of Zhang et al. 2023) using multiple replicate DESI-MSI datasets.

## When NOT to use

- The input is raw, unprocessed imaging data with no per-replicate workspace files available for pseudobulk spectrum construction.
- There is no published or curated reference peak list to compare against, so recovery cannot be quantified.
- The goal is de novo metabolite annotation (e.g., METASPACE or Metlin/LOTUS database searching) rather than benchmarking peak-extraction reproducibility.

## Inputs

- Deposited DESI-MSI replicate datasets (.cdf files plus Matlab .mat workspaces, e.g., B73-root11-workspace.mat, B73-root2-5-workspace.mat, B73-root-3-3 workspace)
- Reference peak list of previously published m/z values (63 consensus peaks from Zhang et al. 2023)

## Outputs

- Per-root mass signature list (CSV) for each of the three B73 replicates
- Pooled unique mass-signature list (CSV) across replicates (301 unique m/z values)
- Recovery percentage against the reference peak list (97% of 63 peaks) plus identification of missed signatures and their abundance/localization characteristics

## How to apply

Load the deposited replicate datasets (e.g., B73-root11, B73-root2-5, B73-root-3-3 .cdf files and their Matlab workspaces from the DIMPLE-code repository). For each replicate, build a pseudobulk mass spectrum by summing per-m/z-bin intensity across all pixels, then extract consensus peaks with a 2.5 ppm mass tolerance. Apply background suppression (retain only peaks exceeding 1.5-fold background intensity) and sparsity filtering to remove m/z signatures present in only a small fraction of pixels. Pool the three per-root signature lists to count unique signatures, then compare against the reference list to compute recovery percentage — counting a reference peak as recovered if a filtered consensus peak falls within the 2.5 ppm window. Save per-root and pooled lists as CSV and verify the reported metrics (average 196 signatures per root, 301 unique, 97% recovery with only 2 misses attributable to very low abundance or sparse localization).

## Related tools

- **DIMPLE** (Pipeline whose consensus-peak extraction (2.5 ppm tolerance, 1.5-fold background suppression, sparsity filtering) is being benchmarked for cross-replicate peak recovery) — github.com/dickinsonlab/DIMPLE-code
- **DESI-MSI** (Source modality supplying the per-pixel mass spectra of the three B73 maize root replicates used for recovery assessment)

## Examples

```
load('B73-root11-workspace.mat')  % then run DIMPLE pseudobulk peak extraction and compare against Zhang et al. 2023 peak list
```

## Evaluation signals

- Average number of m/z signatures per root falls near the reported 196 signatures (three B73 replicates).
- Pooled unique signature count across the three replicates reproduces the reported 301 unique signatures.
- Recovery against the 63 Zhang et al. 2023 peaks reaches approximately 97%, with exactly two missed signatures.
- Missed signatures should be inspectable and attributable to very low abundance or sparse pixel localization.
- All m/z matches to the reference list are within the 2.5 ppm mass tolerance, and all retained peaks exceed 1.5-fold background intensity.

## Limitations

- Recovery is only as good as the reference list; signatures absent from Zhang et al. 2023 cannot be assessed by this comparison.
- Very low-abundance or sparsely localized peaks may fail background suppression or sparsity filtering, as seen with the two unrecovered Zhang et al. 2023 signatures.
- Workspace file naming in the repository is inconsistent (e.g., B73 root 3 workspace is 'B73-root2-3-worskpace.mat', a typo), which can cause load errors if paths are assumed uniform.

## Evidence
