---
name: correlation-range-summary-reporting
description: 'Reference for reproducing blinded biological-replicate correlation analysis:
  you have a SERDA-normalized metabolite intensity matrix plus sample annotations
  identifying which samples are blinded biological duplicates (adjacent or nonadjacent)
  and need to report per-pair and summarized r_xy.'
when_to_use_negative:
- Input is raw, un-normalized metabolite intensities — SERDA normalization must be
  applied first, otherwise replicate correlations reflect instrument drift rather
  than biological reproducibility.
- The task is QC precision (median RSD on pooled or external BioIVT validation QC
  samples) rather than blinded biological-replicate correlation; those use a different
  metric.
- The sample annotations do not identify which samples are blinded biological duplicates
  or whether pairs are adjacent vs. nonadjacent, making the 48/54 split impossible.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Computing per-pair Pearson correlation coefficients across all metabolite
    features and writing the per-pair plus summarized r_xy ranges to CSV.
  repo: https://github.com/slfan2013/SERDA
- name: SERDA
  role: Produces the normalized metabolite intensity matrix on which replicate correlations
    are calculated; runner.R consumes data_file = "filelocation.xlsx".
  repo: https://github.com/slfan2013/SERDA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:561f75400724fad8c8f4993443f90df13730bd8d890d9306a1a11529144f683e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - SERDA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# correlation-range-summary-reporting

## Summary

Quantifies how well SERDA-normalized metabolite intensities reproduce each other across blinded biological duplicate sample pairs by computing per-pair Pearson correlation coefficients (r_xy) over all metabolite features, then summarizing them as an overall r_xy plus adjacent vs. nonadjacent replicate ranges (0.90–1.0 vs. 0.81–1.0) to mirror Supplementary Table S2.

## When to use

Reference for reproducing blinded biological-replicate correlation analysis: you have a SERDA-normalized metabolite intensity matrix plus sample annotations identifying which samples are blinded biological duplicates (adjacent or nonadjacent) and need to report per-pair and summarized r_xy.

## When NOT to use

- Input is raw, un-normalized metabolite intensities — SERDA normalization must be applied first, otherwise replicate correlations reflect instrument drift rather than biological reproducibility.
- The task is QC precision (median RSD on pooled or external BioIVT validation QC samples) rather than blinded biological-replicate correlation; those use a different metric.
- The sample annotations do not identify which samples are blinded biological duplicates or whether pairs are adjacent vs. nonadjacent, making the 48/54 split impossible.

## Inputs

- SERDA-normalized metabolite intensity matrix (e.g. .xlsx file as consumed by SERDA runner.R)
- Sample annotations identifying blinded biological duplicate pairs, distinguishing adjacent (48 pairs) from nonadjacent (54 pairs drawn from sets of 80) replicates

## Outputs

- Per-pair Pearson correlation coefficients (r_xy) across all metabolite features for each of the 102 blinded duplicate pairs
- Summarized overall r_xy (~0.98) plus separate ranges for adjacent (0.90–1.0) and nonadjacent (0.81–1.0) pairs
- CSV table mirroring Supplementary Table S2

## How to apply

Load the SERDA-normalized metabolite intensity matrix in R 3.6.3 (the SERDA runner.R script consumes an .xlsx input file). Identify blinded biological duplicate sample pairs from the sample annotations — in the 4104-sample T2D cohort there were 102 pairs, split into 48 adjacent pairs and 54 nonadjacent pairs drawn from sets of 80. For each duplicate pair, compute the Pearson correlation coefficient (r_xy) across all metabolite features. Aggregate the per-pair correlations into an overall r_xy and report the minimum–maximum range separately for adjacent and nonadjacent pairs (adjacent: 0.90–1.0; nonadjacent: 0.81–1.0; overall r_xy ≈ 0.98). Write the per-pair and summarized results to a CSV table mirroring Supplementary Table S2, and validate that the reproduced overall r_xy and ranges fall within the values shown in Figure 5 / Supplementary Table S2.

## Related tools

- **R** (Computing per-pair Pearson correlation coefficients across all metabolite features and writing the per-pair plus summarized r_xy ranges to CSV.) — https://github.com/slfan2013/SERDA
- **SERDA** (Produces the normalized metabolite intensity matrix on which replicate correlations are calculated; runner.R consumes data_file = "filelocation.xlsx".) — https://github.com/slfan2013/SERDA

## Examples

```
Rscript SERDA_runner.R && Rscript replicate_correlation.R --data_file filelocation.xlsx --duplicates annotations.tsv --out Supplementary_Table_S2.csv
```

## Evaluation signals

- Reproduced overall r_xy should be approximately 0.98 across all metabolites.
- Adjacent duplicate pairs must yield correlations in the range 0.90–1.0.
- Nonadjacent duplicate pairs must yield correlations in the range 0.81–1.0.
- The count of blinded duplicate pairs analyzed should match the sample annotations (48 adjacent + 54 nonadjacent = 102 pairs in the 4104-sample T2D cohort).
- The output CSV should mirror the structure of Supplementary Table S2, with per-pair and summarized rows distinguishable.

## Limitations

- Adjacent vs. nonadjacent replicate adjacency matters — nonadjacent pairs show a wider (lower) minimum correlation (0.81 vs. 0.90), so pooling them without labeling adjacency would obscure the reported difference.
- Results are grounded in a single large GC-MS metabolomics study (>4000 human plasma samples, 1.2 years of acquisition, seven column changes); correlation ranges may differ on smaller or non-plasma datasets.
- The analysis depends on the SERDA-normalized matrix; reproducing it requires the exact SERDA runner.R environment (R 3.6.3, reticulate 1.19, keras 2.7.0, pROC 1.17.0.1, caret 6.0.86, Python 3.6.10).

## Evidence

- [results] Adjacent and nonadjacent replicate correlations both average r_xy = 0.98: [evidence span withheld — anchor status: not_found]
- [results] Adjacent replicate correlation range 0.90–1.0: [evidence span withheld — anchor status: not_found]
- [methods] Workflow: load SERDA-normalized matrix and identify duplicate pairs: [evidence span withheld — anchor status: not_found]
- [readme] Repository implements SERDA normalization: "This is the repository for SERDA normalization"
- [readme] Runner.R consumes xlsx file: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
