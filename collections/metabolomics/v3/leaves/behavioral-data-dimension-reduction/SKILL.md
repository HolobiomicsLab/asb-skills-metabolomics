---
name: behavioral-data-dimension-reduction
description: Use when categorical item scores from a behavioral instrument (e.g. 26
  ASEBA CBCL externalizing items in the NTR or LUMC-Curium cohorts) must be condensed
  into a few interpretable dimensions for downstream integration with omics factors.
when_to_use_negative:
- Input is continuous multi-omics data (DNA methylation or urine metabolomics) — the
  workflow instead uses MOFA2 for those data types
- Data are raw/unstandardized behavioral files; the ACTIONdemonstrator expects inputs
  already converted into standardized file formats by the preprocessing workflow
- The behavioral items are continuous measurements rather than categorical item scores,
  where MCA's categorical association model is inappropriate
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: FactoMineR
  role: R library (v2.4) used to run the MCA on categorical behavioral items and obtain
    behavioral factors and eigenvalues
- name: Nextflow
  role: Workflow manager embedding the MCA step in the modular ACTIONdemonstrator
    analysis pipeline, run with containers for software dependencies
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:b79819bd656cb42a7f84961b88877c6ea1788c1fc0a2be64c3bb32cac0b1a819
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - FactoMineR
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# behavioral-data-dimension-reduction

## Summary

Reduces categorical item-level behavioral questionnaire data (parent-rated ASEBA CBCL externalizing items from the ACTION cohorts) to a small set of orthogonal dimensions via multiple correspondence analysis (MCA) with FactoMineR, so behavioral factors can be correlated with omics factors.

## When to use

Use when categorical item scores from a behavioral instrument (e.g. 26 ASEBA CBCL externalizing items in the NTR or LUMC-Curium cohorts) must be condensed into a few interpretable dimensions for downstream integration with omics factors.

## When NOT to use

- Input is continuous multi-omics data (DNA methylation or urine metabolomics) — the workflow instead uses MOFA2 for those data types
- Data are raw/unstandardized behavioral files; the ACTIONdemonstrator expects inputs already converted into standardized file formats by the preprocessing workflow
- The behavioral items are continuous measurements rather than categorical item scores, where MCA's categorical association model is inappropriate

## Inputs

- Imputed parent-rated externalizing behavior item scores from the ASEBA CBCL (categorical item-level data, e.g. Synthetic_data/synthetic_cbcl_data.csv)
- CBCL item label mapping (Synthetic_data/cbcl_labels.csv)

## Outputs

- MCA dimensions/factors for the behavioral data (dimension scores per subject)
- Per-dimension variance fractions table written to mca_dimension_variance.csv (columns: dimension index, fraction of variation explained)

## How to apply

Load the imputed parent-rated externalizing behavior item scores (CBCL data, e.g. Synthetic_data/synthetic_cbcl_data.csv with item labels from cbcl_labels.csv) into R. Run multiple correspondence analysis (MCA) on the categorical behavioral items with the R library FactoMineR (v2.4), the approach used in the ACTIONdemonstrator workflow to obtain factors for the behavioral data. Extract the eigenvalues from the MCA result and compute the fraction of total variation explained by each dimension. Report the per-dimension variance fractions as a table (dimension index + fraction of variation explained) written to mca_dimension_variance.csv, mirroring the paper's figures: NTR dimensions 1–3 jointly explain 30% of variation across the 26 externalizing items (~16%/9%/<5%), and the LUMC-Curium first 2 dimensions together explain ~30%. The rationale is that MCA suits categorical items and yields behavioral dimension scores that the workflow later compares across SNF clusters and correlates with MOFA omics factors.

## Related tools

- **FactoMineR** (R library (v2.4) used to run the MCA on categorical behavioral items and obtain behavioral factors and eigenvalues)
- **Nextflow** (Workflow manager embedding the MCA step in the modular ACTIONdemonstrator analysis pipeline, run with containers for software dependencies) — https://github.com/Xomics/ACTIONdemonstrator_workflow

## Examples

```
library(FactoMineR); res <- MCA(cbcl_items, graph=FALSE); res$eig
```

## Evaluation signals

- Per-dimension variance fractions for the NTR cohort reproduce the reported values: dimension 1 ~16%, dimension 2 ~9%, dimension 3 <5%, with dimensions 1–3 jointly ~30% of variation across the 26 externalizing CBCL items
- For the LUMC-Curium cohort, the first 2 MCA dimensions together explain ~30% of the variation
- mca_dimension_variance.csv has the expected schema: columns for dimension index and fraction of variation explained
- The number of item variables passed to MCA matches the behavioral item set (26 externalizing behavior items of the ASEBA CBCL for NTR)
- Dimension scores are produced for the same subjects as the omics factors, keyed via the ACTIONdemonstrator ID list

## Limitations

- Privacy regulations prohibit sharing potentially personally identifiable behavioral/clinical data; only a synthetic dataset (Synthetic_data/synthetic_cbcl_data.csv) is provided, so absolute values on real data may differ
- MCA explains only a modest share of total item variation (~30% in the first few dimensions per cohort), so most item-level variation remains in higher dimensions
- The reported reference ranges are cohort-specific (NTR vs LUMC-Curium) and should not be used to validate MCA on other cohorts

## Evidence

- [discussion] MCA applied to behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [discussion] MCA dimension variance in NTR cohort: [evidence span withheld — anchor status: not_found]
- [readme] Behavioral data input file to workflow: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
- [readme] Workflow expects standardized inputs: "This workflows ingests molecular -omics profiling and phenotypic data that is already converted into standardized file formats."
