---
name: pearson-correlation-analysis-of-biological-duplicate-pairs
description: Use when blinded biological replicate (duplicate) samples are embedded
  in a large metabolomics cohort and you need to quantify sample-level reproducibility
  via per-pair Pearson r across all metabolite features, e.g. 102 duplicate pairs
  in a 4104-sample T2D GC-MS plasma study.
when_to_use_negative:
- The input is raw, unnormalized metabolite intensities — correlations on unnormalized
  data will not reflect the reported r_xy ≈ 0.98 because SERDA normalization is what
  brings replicate agreement to the reported range
- The replicate structure is technical (pooled QC) rather than biological duplicates
  pooled QC samples are handled by the separate SERDA/SERRF precision assessment (median
  16% training RSD, 19% independent BioIVT RSD)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Computes per-pair Pearson correlation coefficients over the 102 blinded biological
    duplicate pairs and writes the Supplementary Table S2-style CSV
  repo: github.com/slfan2013/SERDA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:8ed113e824ae7d38cead1a5f78151beb600ac324009aae5edc3c8f58731fd93a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pearson-correlation-analysis-of-biological-duplicate-pairs

## Summary

Quantifies reproducibility of blinded biological duplicate pairs in large-scale GC-MS metabolomics by computing Pearson correlation coefficients (r_xy) across all metabolite features for each duplicate pair. Applied to the 4104-sample type 2 diabetes cohort's 102 duplicate pairs to validate SERDA-normalized intensities, distinguishing adjacent from nonadjacent replicates.

## When to use

Use when blinded biological replicate (duplicate) samples are embedded in a large metabolomics cohort and you need to quantify sample-level reproducibility via per-pair Pearson r across all metabolite features, e.g. 102 duplicate pairs in a 4104-sample T2D GC-MS plasma study.

## When NOT to use

- The input is raw, unnormalized metabolite intensities — correlations on unnormalized data will not reflect the reported r_xy ≈ 0.98 because SERDA normalization is what brings replicate agreement to the reported range
- The replicate structure is technical (pooled QC) rather than biological duplicates pooled QC samples are handled by the separate SERDA/SERRF precision assessment (median 16% training RSD, 19% independent BioIVT RSD)

## Inputs

- SERDA-normalized metabolite intensity matrix (xlsx) from the >4000 human plasma samples in the type 2 diabetes cohort study
- Sample annotations identifying 102 blinded biological duplicate pairs (48 adjacent, 54 nonadjacent)

## Outputs

- Per-pair Pearson correlation coefficients (r_xy) across all metabolite features for each of the 102 duplicate pairs
- Grouped summary statistics: overall r_xy ≈ 0.98 with ranges 0.90–1.0 (adjacent) and 0.81–1.0 (nonadjacent)
- CSV table mirroring Supplementary Table S2

## How to apply

Load the SERDA-normalized metabolite intensity matrix in R 3.6.3 (run `SERDA runner.R` with `data_file = "filelocation.xlsx"` after configuring reticulate 1.19, keras 2.7.0, pROC 1.17.0.1, caret 6.0.86, and Python 3.6.10 via r-miniconda). Identify the 102 blinded biological duplicate sample pairs from sample annotations and split them into 48 adjacent pairs versus 54 nonadjacent pairs drawn from sets of 80. Compute the Pearson correlation coefficient (r_xy) across all 563 metabolite annotations for each duplicate pair independently, then aggregate per-group statistics. Report the overall r_xy (expected ≈ 0.98) plus the observed range for adjacent pairs (0.90–1.0) and nonadjacent pairs (0.81–1.0), and write per-pair and summarized results to a CSV table mirroring Supplementary Table S2.

## Related tools

- **R** (Computes per-pair Pearson correlation coefficients over the 102 blinded biological duplicate pairs and writes the Supplementary Table S2-style CSV) — github.com/slfan2013/SERDA

## Examples

```
Rscript SERDA_runner.R  # then in R: pairs <- readRDS('dup_pairs.rds'); r <- sapply(pairs, function(p) cor(mat[p[1],], mat[p[2],])); write.csv(r, 'SuppTable_S2.csv')
```

## Evaluation signals

- Overall aggregated Pearson r_xy should be ≈ 0.98 across all metabolites
- Adjacent duplicate-pair correlations must fall within 0.90–1.0 and nonadjacent pairs within 0.81–1.0 (Figure 5 / Supplementary Table S2 ranges)
- Pair count check: exactly 102 blinded biological duplicate pairs identified (48 adjacent + 54 nonadjacent)
- Feature coverage: correlation computed over all metabolite features in the SERDA-normalized matrix, consistent with the 563 metabolite annotations

## Limitations

- Reproducibility depends on SERDA normalization quality — using raw or sum-based/internal-standard-normalized intensities gives median precision above 30% RSD and would depress replicate correlations
- Adjacent and nonadjacent replicate classes have different correlation floors (0.90 vs 0.81), so replicates must be correctly classified before aggregation
- The reported r_xy ≈ 0.98 was established for the 4104-sample T2D plasma GC-MS cohort; transferring thresholds to other cohorts or platforms is not directly validated
- Requires correctly annotated blinded duplicate labels; mislabeled or missing annotations will corrupt pair identification

## Evidence

- [full_text] Adjacent vs nonadjacent replicate classification: [evidence span withheld — anchor status: not_found]
- [full_text] Reported correlation ranges: [evidence span withheld — anchor status: not_found]
- [full_text] Duplicate pair identification workflow: [evidence span withheld — anchor status: not_found]
- [full_text] Pairwise correlation computation: [evidence span withheld — anchor status: not_found]
- [full_text] Output table format: [evidence span withheld — anchor status: not_found]
- [readme] SERDA runner execution: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [abstract] Normalization context for interpretability: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
