---
name: opls-da-loading-extraction-and-ranking
description: Use when a two-group lipidomics contrast needs a ranked list of discriminating
  lipids — e.g. extracting the top 10 OPLS-DA loadings separating Benign from Cancer
  samples in a Metabolomics Workbench study like ST001111.
when_to_use_negative:
- The contrast is univariate or design-based (use lipidr's differential analysis /
  volcano plots instead)
- The dataset has no reliable sample-group annotation, so no supervised OPLS-DA model
  can be fit
- Lipid names cannot be parsed into class/chain information, so top_lipids annotation
  would be unreliable
tools:
- name: lipidr
  role: 'Provides the LipidomicsExperiment container and the OPLS-DA workflow: mva,
    plot_mva_loadings, and top_lipids for ranking and annotating top lipids'
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment; lipidr is an open-source R/Bioconductor package installable
    via BiocManager
- name: Metabolomics Workbench API
  role: Source of public lipidomics datasets (e.g. ST001111) for search, download,
    and reanalysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:6e623e51efe94269ab2e9f25959e489d90bc2a11aa1ea1dc8c8c00ffd77204a3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R/Bioconductor
  - lipidr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# opls-da-loading-extraction-and-ranking

## Summary

Ranks lipids by their contribution to supervised OPLS-DA class separation using lipidr's mva/plot_mva_loadings/top_lipids functions, as demonstrated on the ST001111 breast cancer lipidomics dataset (Benign vs Cancer).

## When to use

Use when a two-group lipidomics contrast needs a ranked list of discriminating lipids — e.g. extracting the top 10 OPLS-DA loadings separating Benign from Cancer samples in a Metabolomics Workbench study like ST001111.

## When NOT to use

- The contrast is univariate or design-based (use lipidr's differential analysis / volcano plots instead)
- The dataset has no reliable sample-group annotation, so no supervised OPLS-DA model can be fit
- Lipid names cannot be parsed into class/chain information, so top_lipids annotation would be unreliable

## Inputs

- LipidomicsExperiment object with a numeric lipid-by-sample intensity matrix and sample annotations (grouping column such as SampleType)
- Metabolomics Workbench study files (e.g. ST001111) or a Skyline-exported CSV plus a sample-annotation CSV

## Outputs

- OPLS-DA loadings plot colored by lipid class
- Exactly-N-row (e.g. 10) ranked table of top lipids with lipid name and lipid class annotation, e.g. top10_oplsda_lipids.csv

## How to apply

Load the cleaned lipidomics dataset into lipidr's LipidomicsExperiment container (e.g. via as_lipidomics_experiment or read_skyline plus add_sample_annotation), ensuring the grouping column (e.g. SampleType) is present. Fit a supervised discriminant model restricted to the two classes of interest with mva(method="OPLS-DA", group_col=..., groups=c(...)), since the OPLS-DA model must be trained on exactly the two groups being contrasted. Extract the model loadings with plot_mva_loadings and rank lipids by the magnitude of their contribution to class separation; loadings can be colored by lipid class. Retrieve class/chain annotations for the top-ranked features with top_lipids (e.g. top.n=10), which leverages lipidr's automatic inference of lipid class and chain information from lipid names. Write the ranked, annotated table to CSV (e.g. top10_oplsda_lipids.csv).

## Related tools

- **lipidr** (Provides the LipidomicsExperiment container and the OPLS-DA workflow: mva, plot_mva_loadings, and top_lipids for ranking and annotating top lipids) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment; lipidr is an open-source R/Bioconductor package installable via BiocManager)
- **Metabolomics Workbench API** (Source of public lipidomics datasets (e.g. ST001111) for search, download, and reanalysis)

## Examples

```
d <- lipidr::mva(d, method = "OPLS-DA", group_col = "SampleType", groups = c("Benign","Cancer"))
```

## Evaluation signals

- Output table contains exactly 10 rows (matching top.n=10)
- Every output row carries both a lipid name and a lipid class annotation
- Rows are ordered by the OPLS-DA loading ranking (magnitude of contribution to class separation)
- The OPLS-DA model was fit only on the two groups of interest (Benign and Cancer), not the full sample set
- Annotations are consistent with lipidr's inference of lipid class and chain information from lipid names

## Limitations

- top_lipids annotation depends on automatic inference from lipid names; non-standard lipid names may yield missing or wrong class annotations
- OPLS-DA is supervised and two-class; the model must be restricted to the two groups being contrasted (Benign vs Cancer here), otherwise the loading ranking is not meaningful for the target contrast
- Only PCA, PCoA and OPLS(DA) multivariate methods are implemented in lipidr

## Evidence

- [readme] OPLS-DA implementation and interactive loadings exploration: [evidence span withheld — anchor status: not_found]
- [readme] Top associated lipids and loadings plots are inspectable: [evidence span withheld — anchor status: external_doc]
- [readme] Sample annotation must be attached to the experiment: [evidence span withheld — anchor status: external_doc]
