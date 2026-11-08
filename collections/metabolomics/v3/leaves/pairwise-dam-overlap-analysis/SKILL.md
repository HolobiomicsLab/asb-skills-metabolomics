---
name: pairwise-dam-overlap-analysis
description: Use when several normalization methods have been applied to the same
  metabolomics dataset and you need to quantify consistency of their differential
  abundance calls (e.g. DAM overlap and p-value similarity across methods).
when_to_use_negative:
- When the question is whether normalization improved replicate concordance or removed
  drift/batch effects — use QC RSD, pre/post normalization plots, and PCA score plots
  instead; this skill addresses downstream DAM consistency, not normalization performance
- When multiple testing correction is required for a definitive differential abundance
  result — this workflow uses unadjusted two-sided Wilcoxon tests at p < 0.05 specifically
  to compare method agreement, not to make final DAM claims
- When the dataset has no repeated measurements under multiple normalization strategies
  or no biological contrast to test — overlap and p-value similarity are undefined
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: Metanorm R package
  role: Provides the batchwise normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM,
    tGAM) whose normalized outputs feed the per-method Wilcoxon DAM calls being compared
  repo: https://github.com/UGent-LIMET/Metanorm
- name: R
  role: Environment for running Metanorm normalization, Wilcoxon rank sum tests (p
    < 0.05), Spearman correlations, Euclidean distance computation, and complete-linkage
    heatmap generation
- name: TIGER
  role: Multivariate normalization method run via its R package to generate one of
    the DAM sets included in the pairwise overlap comparison
- name: SERFF
  role: Multivariate normalization method run via its Shiny interface to generate
    one of the DAM sets included in the pairwise overlap comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:4b532b1b0affd41ac670d565e16d5e68521ebfd796ab4150f614886465924299
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
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# pairwise-dam-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how strongly different normalization methods (robust rLOESS/rGAM/tGAM vs. QC-RLSC, QC-RSC, SERFF, TIGER) agree in the differentially abundant metabolites (DAMs) they call, using pairwise DAM overlap percentages plus p-value correlation-based clustering. It was used to show that robust methods agree 74–88% while QC-RLSC agrees only 47–51% on the ENVIRONAGE sex contrast.

## When to use

Use when several normalization methods have been applied to the same metabolomics dataset and you need to quantify consistency of their differential abundance calls (e.g. DAM overlap and p-value similarity across methods).

## When NOT to use

- When the question is whether normalization improved replicate concordance or removed drift/batch effects — use QC RSD, pre/post normalization plots, and PCA score plots instead; this skill addresses downstream DAM consistency, not normalization performance
- When multiple testing correction is required for a definitive differential abundance result — this workflow uses unadjusted two-sided Wilcoxon tests at p < 0.05 specifically to compare method agreement, not to make final DAM claims
- When the dataset has no repeated measurements under multiple normalization strategies or no biological contrast to test — overlap and p-value similarity are undefined

## Inputs

- Normalized metabolite intensity matrices (rows = compounds/metabolites, columns = sample runs) from each normalization method (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC via Metanorm batchwise mode; SERFF; TIGER)
- Sample metadata: batch assignment vector and sample type vector ('QC' vs. other), plus the biological contrast variable (e.g. sex)
- Log-transformed (base e) raw intensities, with the 47 samples occurring before the first QC in each batch excluded for QC-RLSC/QC-RSC normalization

## Outputs

- Pairwise DAM overlap percentages between normalization methods (expected ranges: robust 74–88%, QC-RLSC 47–51%, SERFF 68–70%, TIGER 64–67%)
- Pairwise Spearman correlation matrix of method p-value vectors
- Heatmap of p-value similarity (Euclidean distances of one-minus-correlation vectors) with complete-linkage hierarchical clustering
- Summary table (e.g. environage_dam_comparison.csv) with all overlap percentages

## How to apply

For each normalized dataset (produced by Metanorm with model='QC-RLSC', 'QC-RSC', 'rLOESS', 'rGAM', 'tGAM' in batchwise mode, plus SERFF and TIGER), run two-sided Wilcoxon rank sum tests on the contrast of interest (e.g. biological sex) at p < 0.05 to obtain each method's DAM set. Compute pairwise DAM overlap percentages between all normalization methods, and separately compute pairwise Spearman correlations of the methods' full p-value vectors. Convert correlation vectors to dissimilarity via one minus correlation, compute pairwise Euclidean distances on those vectors, and build a heatmap ordered by complete-linkage hierarchical clustering. In the ENVIRONAGE application (393 runs, 269 metabolites, 6 batches), robust methods showed 74–88% DAM overlap, QC-RLSC 47–51%, SERFF 68–70%, and TIGER 64–67%, demonstrating that normalization choice materially changes downstream DAM detection. Note the paper explicitly does not apply multiple testing correction at this p < 0.05 stage, flagging it as essential for downstream statistical inference in general.

## Related tools

- **Metanorm R package** (Provides the batchwise normalization methods (QC-RLSC, QC-RSC, rLOESS, rGAM, tGAM) whose normalized outputs feed the per-method Wilcoxon DAM calls being compared) — https://github.com/UGent-LIMET/Metanorm
- **R** (Environment for running Metanorm normalization, Wilcoxon rank sum tests (p < 0.05), Spearman correlations, Euclidean distance computation, and complete-linkage heatmap generation)
- **TIGER** (Multivariate normalization method run via its R package to generate one of the DAM sets included in the pairwise overlap comparison)
- **SERFF** (Multivariate normalization method run via its Shiny interface to generate one of the DAM sets included in the pairwise overlap comparison)

## Examples

```
normdat <- metanorm(rawdata[1:5,], model = "tGAM", type = metanorm.qc, QCcheck = TRUE, batch = batch, plotdir = "~/Documents/metanormExample/")
```

## Evaluation signals

- Reproduced robust-method pairwise DAM overlaps fall within the reported 74–88% range
- QC-RLSC overlap percentages fall within the reported 47–51% range (and SERFF 68–70%, TIGER 64–67%)
- Per-method DAM lists are derived from two-sided Wilcoxon rank sum tests at p < 0.05 on the sex contrast for every normalized dataset
- Heatmap ordering is produced by complete-linkage hierarchical clustering on Euclidean distances of one-minus-Spearman-correlation vectors, and the heatmap file is generated
- All overlap percentages are persisted to the comparison CSV (e.g. environage_dam_comparison.csv) with one row per method pair

## Limitations

- DAM calls use unadjusted p < 0.05; the authors state multiple testing correction is essential for downstream statistical inference but was out of scope here
- Benchmarking covered only the listed methods — broader sets of normalization methods, and formal benchmarking of the multivariate methods SERFF and TIGER, remained as future work
- Neutrality in benchmarking (not favoring any method, e.g. via QC RSD reduction or increased DAM counts as proxies) requires care and is flagged as a concern by the authors
- QC-based methods require excluding samples occurring before the first QC in each batch (47 samples in ENVIRONAGE), so compared datasets are not identically sized across methods

## Evidence

- [readme] The R package implements three (new) robust normalization methods (tGAM, rGAM and rLOESS), alongside formerly proposed ones (QC-RLSC, QC-RSC).: [evidence span withheld — anchor status: external_doc]
