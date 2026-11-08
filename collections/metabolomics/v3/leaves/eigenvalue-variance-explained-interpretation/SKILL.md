---
name: eigenvalue-variance-explained-interpretation
description: Use when MCA (or a related eigen-decomposition method) has been run on
  categorical behavioral items and you need to quantify and report how much variation
  each dimension explains.
when_to_use_negative:
- The data are continuous omics measurements (DNA methylation, metabolomics) that
  the workflow factors with MOFA2 instead of MCA
- You need cluster assignments or integrated similarity networks — that is the downstream
  SNF/SNFpy eigengap clustering step, not eigenvalue reporting
- The behavioral items are already reduced to dimension scores and only require group
  comparisons across SNF clusters
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FactoMineR
  role: R library (v2.4) used to run the MCA on the behavioral data whose eigenvalues/variance
    fractions are extracted and interpreted
- name: Nextflow
  role: Workflow manager wrapping the MCA step and the eigenvalue/variance reporting
    step (output mca_dimension_variance.csv) in the ACTIONdemonstrator pipeline
  repo: https://github.com/Xomics/ACTIONdemonstrator_workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad115
    title: A multi-omics data analysis workflow packaged as a FAIR Digital Object
  source_content_hash: sha256:a900e277438dcba17529cab13e440023a244b231fa05f2a127d282c627c59339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xomics/ACTIONdemonstrator_workflow
  tools:
  - FactoMineR
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# eigenvalue-variance-explained-interpretation

## Summary

Extract and interpret eigenvalues from a multiple correspondence analysis (MCA) of parent-rated ASEBA CBCL externalizing behavior items, converting them into per-dimension fractions of total variation to identify how many behavioral dimensions carry signal.

## When to use

Use when MCA (or a related eigen-decomposition method) has been run on categorical behavioral items and you need to quantify and report how much variation each dimension explains.

## When NOT to use

- The data are continuous omics measurements (DNA methylation, metabolomics) that the workflow factors with MOFA2 instead of MCA
- You need cluster assignments or integrated similarity networks — that is the downstream SNF/SNFpy eigengap clustering step, not eigenvalue reporting
- The behavioral items are already reduced to dimension scores and only require group comparisons across SNF clusters

## Inputs

- Categorical parent-rated externalizing behavior item scores from the ASEBA CBCL (e.g., Synthetic_data/synthetic_cbcl_data.csv, with labels from Synthetic_data/cbcl_labels.csv)
- MCA result object from FactoMineR (v2.4) fitted on the behavioral items

## Outputs

- Table of dimension index and fraction of total variation explained per MCA dimension
- mca_dimension_variance.csv
- Interpretation of the leading dimensions (e.g., NTR dim 1 ~16%, dim 2 ~9%, dims 1-3 jointly 30%) mirroring the paper's figures

## How to apply

After running MCA on the categorical parent-rated externalizing behavior items of the ASEBA CBCL with the R library FactoMineR (v2.4), extract the eigenvalues from the MCA result object and divide each dimension's eigenvalue by the total sum to compute the fraction of total variation explained. In the ACTION case study this revealed that in the NTR cohort the first 3 MCA dimensions jointly explain 30% of the variation in 26 externalizing behavior items, with dimension 1 explaining ~16%, dimension 2 ~9%, and dimension 3 <5%. Report the results as a table with columns for the dimension index and the fraction of variation explained, and write it to mca_dimension_variance.csv so downstream steps (e.g., comparing MCA behavioral dimension scores across SNF clusters) can consume the dimensions.

## Related tools

- **FactoMineR** (R library (v2.4) used to run the MCA on the behavioral data whose eigenvalues/variance fractions are extracted and interpreted)
- **Nextflow** (Workflow manager wrapping the MCA step and the eigenvalue/variance reporting step (output mca_dimension_variance.csv) in the ACTIONdemonstrator pipeline) — https://github.com/Xomics/ACTIONdemonstrator_workflow

## Examples

```
Rscript -e 'library(FactoMineR); mca <- MCA(read.csv("Synthetic_data/synthetic_cbcl_data.csv"), graph=FALSE); frac <- mca$eig[,2]/sum(mca$eig[,1]); write.csv(data.frame(dimension=seq_along(frac), variance_fraction=frac), "mca_dimension_variance.csv", row.names=FALSE)'
```

## Evaluation signals

- Per-dimension variance fractions reproduce the reported NTR figures: dimension 1 ~16%, dimension 2 ~9%, dimension 3 <5%
- The first 3 NTR dimensions jointly explain ~30% of the variation in the 26 externalizing behavior items
- For the LUMC-Curium cohort, the first 2 dimensions together explain ~30% of the variation
- Output file mca_dimension_variance.csv exists and has columns for dimension index and fraction of variation explained
- Fractions across all dimensions sum to 1 (each eigenvalue divided by the total)

## Limitations

- Only a synthetic dataset is shared because privacy regulations prohibit sharing potentially personally identifiable data, so variance fractions on synthetic data will not match the reported cohort figures exactly
- MCA dimensions each explain modest variance (<5% beyond dimension 3 in NTR), so dimension selection cutoffs must be chosen carefully rather than relying on a single dominant eigenvalue

## Evidence

- [discussion] MCA applied to behavioral data: "applied MCA [65] using the R library FactoMineR (2.4) [64] to obtain factors for the behavioral data"
- [other] Source card finding on variance explained: [evidence span withheld — anchor status: not_found]
- [readme] Behavioral data input file: "--behavioral_data Synthetic_data/synthetic_cbcl_data.csv"
