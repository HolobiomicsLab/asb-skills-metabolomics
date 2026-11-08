---
name: lc-ms-batch-effect-quantification
description: Use when LC-MS metabolomics data contain identical biological samples
  run in subsequent batches (between-batch replicate pairs) and you need to quantify
  which normalization restores cross-batch intensity correspondence.
when_to_use_negative:
- Data contain only pooled QC samples and no biological replicate pairs run across
  batches — replicate concordance cannot be computed, and the article explicitly flags
  reduction of QC-sample relative standard deviation as a benchmarking pitfall proxy
- Goal is differential abundance testing rather than quantifying cross-batch replicate
  correspondence; the article shows normalization choice produces distinct differential
  abundance results, which is a separate downstream analysis
- Input is a single-batch data set with no between-batch replicate structure, so the
  rescaled-difference MAD metric cannot be defined
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metanorm R package
  role: Provides the normalization implementations (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC)
    whose Metanorm-processed outputs are compared via the replicate MAD concordance
    metric
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Environment in which the MAD of rescaled replicate differences is computed
    per normalization approach and the percentage reductions are tabulated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:ca62cea9c36acdcc91e9fc2574822e6b93ce95004c2f2610fd6a635e6b264048
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lc-ms-batch-effect-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies residual between-batch LC-MS intensity differences by computing the MAD of rescaled intensity differences between between-batch replicate pairs under each normalization approach, benchmarking robust methods (tGAM, rGAM, rLOESS) against QC-based and multivariate normalizers and an unnormalized baseline.

## When to use

Use when LC-MS metabolomics data contain identical biological samples run in subsequent batches (between-batch replicate pairs) and you need to quantify which normalization best restores cross-batch intensity correspondence.

## When NOT to use

- Data contain only pooled QC samples and no biological replicate pairs run across batches — replicate concordance cannot be computed, and the article explicitly flags reduction of QC-sample relative standard deviation as a benchmarking pitfall proxy
- Goal is differential abundance testing rather than quantifying cross-batch replicate correspondence; the article shows normalization choice produces distinct differential abundance results, which is a separate downstream analysis
- Input is a single-batch data set with no between-batch replicate structure, so the rescaled-difference MAD metric cannot be defined

## Inputs

- BioHEART between-batch replicate-run intensity matrix: pairs of identical biological samples run in subsequent batches
- Normalized intensities from each tested method: tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, QC-RSC (Metanorm-processed outputs)
- Unnormalized intensities (log-transformed, base e, prior to normalization) as baseline

## Outputs

- replicate_concordance_mad.tsv — per-method MAD of rescaled replicate differences and percentage reduction versus unnormalized baseline
- Per-method ranking of cross-batch replicate correspondence (lower MAD = better correspondence)

## How to apply

Load the BioHEART replicate-run intensity data — pairs of identical biological samples run in subsequent batches — with normalized intensities from tGAM, rGAM, rLOESS, SERFF, TIGER, QC-RLSC, and QC-RSC plus unnormalized intensities, all log-transformed (base e) prior to normalization. For each replicate pair and compound, calculate the difference between the two observed intensities rescaled by their mean intensity. Compute the MAD of these rescaled differences per normalization approach in R using Metanorm-processed outputs, then compute each method's percentage reduction in MAD versus the unnormalized baseline. Lower MAD indicates better cross-batch correspondence and superior normalization; tabulate per-method MAD and % reduction and save as replicate_concordance_mad.tsv. Validate against reported values: robust methods 49–52% reduction, SERFF 41%, TIGER 40%, QC-RLSC/QC-RSC 18–22%.

## Related tools

- **Metanorm R package** (Provides the normalization implementations (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC) whose Metanorm-processed outputs are compared via the replicate MAD concordance metric) — https://github.com/UGent-LIMET/Metanorm
- **R** (Environment in which the MAD of rescaled replicate differences is computed per normalization approach and the percentage reductions are tabulated)

## Examples

```
normdat <- metanorm(rawdata, model="tGAM", type=metanorm.qc, batch=batch, QCcheck=TRUE, plotdir="~/metanormOut/")
```

## Evaluation signals

- Robust methods (tGAM, rGAM, rLOESS) show a 49–52% MAD reduction versus unnormalized data
- SERFF shows 41% and TIGER 40% reduction; QC-RLSC and QC-RSC show only 18–22%, matching reported values
- Each method's relative MAD reduction is computed against the unnormalized baseline on identical replicate pairs and compounds
- All intensities are log-transformed (base e) before normalization; differences between replicate intensities are rescaled by their mean intensity before MAD computation
- Output table replicate_concordance_mad.tsv contains per-method MAD and % reduction with the robust methods ranked lowest (best cross-batch correspondence)

## Limitations

- The metric evaluates feature-level (compound-level) behavior; multivariate outlier treatment is not implemented in Metanorm
- Unexpected differences between QC samples and biological samples require analyst scrutiny rather than automated handling — Metanorm can flag QC discrepancies but judgment is manual
- Automated outlier detection and removal prior to normalization is nontrivial and not provided; QC-based methods (QC-RLSC, QC-RSC) can be deteriorated by occasional disproportionately high or low outlier intensities
- Cross-metabolite information is not leveraged by metabolite-level normalization methods, which may limit residual batch-effect removal for correlated compounds

## Evidence

- [readme] PCA batch-effect verification via Metanorm: [evidence span withheld — anchor status: external_doc]
- [readme] Visual inspection catches subpar normalization despite clean PCA: [evidence span withheld — anchor status: external_doc]
