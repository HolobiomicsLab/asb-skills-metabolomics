---
name: tissue-separation-verification-in-ordination-space
description: Use when a labeled metabolomics feature table (samples x features with
  tissue/group labels) needs a check that one group forms a distinct cluster in unsupervised
  PCA space, e.g. buccal gland vs. other tissues.
when_to_use_negative:
- The input is raw instrument data (e.g. mzML/NetCDF) rather than a processed feature
  table — peak picking and alignment must precede PCA
- A supervised group-difference test is required — PCA is unsupervised and only demonstrates
  visual/structural separation, not statistical significance
- The question concerns feature-level relationships (e.g. metabolite annotation or
  clustering) rather than sample-level group separation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: Interactive R Shiny app providing the data preprocessing, PCA/statistical
    analysis, visualization, and self-contained HTML reporting used to generate and
    verify the score plot
  repo: https://github.com/YonghuiDong/MetaboReport
- name: stats
  role: R's stats::prcomp performs the principal component analysis on the autoscaled
    feature matrix
- name: ggplot2
  role: Draws the PC1-vs-PC2 score plot with points colored by tissue and exports
    it as PNG
- name: R
  role: Runtime environment in which MetaboReport (installable as an R package) and
    the analysis scripts execute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:36abe1340f398f7fd07b9ddc1dec42d47b2cd4594ec846a59d141a05f96c9432
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - ggplot2
  - ropls
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tissue-separation-verification-in-ordination-space

## Summary

Verify that a tissue of interest separates from other tissues in ordination space by running PCA on a variance-stabilized, autoscaled LC-MS feature table (e.g. the lamprey spatial metabolomics dataset, 94 samples across 14 tissues, negative ion mode) and inspecting the PC1-vs-PC2 score plot colored by tissue, as demonstrated with MetaboReport.

## When to use

Use when a labeled metabolomics feature table (samples x features with tissue/group labels) needs a check that one group forms a distinct cluster in unsupervised PCA space, e.g. buccal gland vs. other tissues.

## When NOT to use

- The input is raw instrument data (e.g. mzML/NetCDF) rather than a processed feature table — peak picking and alignment must precede PCA
- A supervised group-difference test is required — PCA is unsupervised and only demonstrates visual/structural separation, not statistical significance
- The question concerns feature-level relationships (e.g. metabolite annotation or clustering) rather than sample-level group separation

## Inputs

- LC-MS feature intensity table (samples x features), e.g. Supplementary Information File S4 from the lamprey spatial metabolomics study (94 samples, 14 tissues, negative ion mode)
- Per-sample tissue/group label vector

## Outputs

- Per-sample PC1/PC2 scores table saved as CSV
- PC1-vs-PC2 PCA score plot (PNG) with points colored by tissue

## How to apply

Load the feature table (for the lamprey case, Supplementary Information File S4: 94 samples across 14 tissues, negative ion mode LC-MS) together with the tissue label per sample. Apply a variance-stabilizing transformation to feature intensities, then autoscale each feature (mean-center, unit-variance) so high-abundance ions do not dominate the PCA. Run PCA on the scaled matrix (stats::prcomp via MetaboReport's internal analysis functions), extract per-sample PC1 and PC2 coordinates into a CSV scores table, and draw the PC1-vs-PC2 score plot with ggplot2 coloring points by tissue. The skill succeeds when the target tissue's samples cluster apart from all other groups, reproducing the separation reported in the article (buccal gland distinctly separated from the other 13 tissues).

## Related tools

- **MetaboReport** (Interactive R Shiny app providing the data preprocessing, PCA/statistical analysis, visualization, and self-contained HTML reporting used to generate and verify the score plot) — https://github.com/YonghuiDong/MetaboReport
- **stats** (R's stats::prcomp performs the principal component analysis on the autoscaled feature matrix)
- **ggplot2** (Draws the PC1-vs-PC2 score plot with points colored by tissue and exports it as PNG)
- **R** (Runtime environment in which MetaboReport (installable as an R package) and the analysis scripts execute)

## Examples

```
library(MetaboReport); run_app()  # then load feature table + tissue labels, run PCA module, export PC1/PC2 scores CSV and ggplot2 score plot PNG
```

## Evaluation signals

- The target tissue's samples form a coherent cluster separated from all other tissues in the PC1–PC2 score plot (for the lamprey data: buccal gland separated from the other 13 tissues)
- The extracted PC scores table contains one PC1/PC2 pair per sample (94 rows for the lamprey S4 dataset)
- Autoscaling was applied (each feature mean-centered with unit variance) — unscaled or only log-transformed data should not be used directly in prcomp
- The reproduced plot matches the tissue separation reported in Section 3 of the source article
- MetaboReport's HTML report includes the interactive score plot and statistical results alongside the exported PNG/CSV artifacts

## Limitations

- PCA separation is qualitative — it demonstrates a unique metabolic profile visually but does not by itself provide a statistical significance test of group differences
- High-abundance ions dominate the projection unless features are variance-stabilized and autoscaled (mean-center, unit-variance) beforehand
- MetaboReport's public package API is intentionally limited to run_app(); analysis helpers are internal, so programmatic reuse depends on the app's UI/modules rather than a stable function contract

## Evidence

- [supplementary] Apply a variance-stabilizing transformation to the feature intensities and autoscale (mean-center, unit-variance) each feature so high-abundance ions do not dominate: [evidence span withheld — anchor status: not_found]
- [results] PCA separation finding: "The PCA score plot generated by MetaboReport clearly shows that the buccal gland is distinctly separated from all other tissues, indicating a unique metabolic profile."
- [intro] Shiny app scope: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [readme] Internal API limitation: "The supported public package API is intentionally small: `run_app()`. Analysis helpers are internal so their implementation can evolve"
