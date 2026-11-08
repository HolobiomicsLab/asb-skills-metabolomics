---
name: multi-omics-unit-variance-standardisation
description: Use when molecular abundance matrices from different omics layers (e.g.
  ChEBI-annotated metabolomics, UniProt proteomics, ENSEMBL transcriptomics) must
  be brought to a common scale prior to single-sample pathway analysis or pathway-based
  multi-omics model fitting.
when_to_use_negative:
- Data are already pathway-transformed (an N × P pathway-score table) — re-scaling
  pathway scores is not part of this skill
- Individuals lack measurements for one or more omics views, since PathIntegrate requires
  complete multi-omics data and would force sample discarding; use a method tolerant
  of missing views instead
- A substantial fraction of molecules cannot be mapped to pathway-database identifiers,
  since un-annotated molecules cannot be incorporated into pathway-based models and
  cause information loss
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: sspa
  role: Python package whose README documents the z-score standardisation step and
    provides the pathway scoring (sspa_kpca, sspa_SVD) that consumes the scaled matrices
  repo: github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Applies unit-variance standardisation automatically to each input omics matrix
    before pathway transformation
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: sspa methods expose fit/transform/fit_transform for compatibility with SKlearn
    Pipeline and GridSearchCV, so scaled matrices can flow through standard ML workflows
- name: Python
  role: Implementation language for the pandas-based log-transform, median imputation
    and column-wise z-score scaling shown in the README
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:45102fa0353f3dbf30e6a0a9a624c7c482ce0f431f9221546535cb81348dffeb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Multi-omics unit-variance standardisation

## Summary

Z-score (unit-variance) scaling of each molecular abundance matrix — metabolomics, proteomics, transcriptomics — so that downstream sspa kPCA/PLAGE pathway scoring brings disparate omics datasets to a common scale before supervised integration modelling in PathIntegrate.

## When to use

Use when molecular abundance matrices from different omics layers (e.g. ChEBI-annotated metabolomics, UniProt proteomics, ENSEMBL transcriptomics) must be brought to a common scale prior to single-sample pathway analysis or pathway-based multi-omics model fitting.

## When NOT to use

- Data are already pathway-transformed (an N × P pathway-score table) — re-scaling pathway scores is not part of this skill
- Individuals lack measurements for one or more omics views, since PathIntegrate requires complete multi-omics data and would force sample discarding; use a method tolerant of missing views instead
- A substantial fraction of molecules cannot be mapped to pathway-database identifiers, since un-annotated molecules cannot be incorporated into pathway-based models and cause information loss

## Inputs

- Molecular abundance matrix as a pandas DataFrame (samples × molecules) per omics layer, with column identifiers matching the pathway database (ChEBI/UniProt/ENSEMBL for Reactome, KEGG compound/gene IDs for KEGG)
- Pathway DataFrame from the sspa package (pathways × molecules)
- Metadata columns (e.g. outcome group) excluded from the values used for scaling

## Outputs

- Unit-variance (z-scored) molecular abundance matrix per omics layer (mean=0, std=1 per molecule column)
- N × P pathway activity-score matrix per omics layer (index = sample IDs, columns = pathway IDs) after downstream sspa kPCA/PLAGE scoring

## How to apply

For each omics matrix, log-transform the raw abundances first (the README recommends np.log2 after median imputation for the COVID-19 metabolomics data), then z-score each molecule column by subtracting the column mean and dividing by the column standard deviation: (X - X.mean(axis=0)) / X.std(axis=0), producing mean=0 and std=1. PathIntegrate performs this unit-variance standardisation automatically on its inputs, so the practitioner must ensure identifiers (ChEBI, UniProt, ENSEMBL for Reactome; KEGG IDs for KEGG) already match the chosen pathway database before scaling. Columns with excessive missingness — the sspa README example filters metabolites where the proportion of NA/0/' ' values exceeds 0.5 — should be removed and remaining gaps median-imputed before scaling. Decision point: metabolite columns that fail identifier mapping to the pathway database are dropped rather than scaled, and samples missing entire omics views must be discarded because PathIntegrate does not support whole-row missingness. After scaling, each N-by-M matrix feeds sspa.sspa_kpca or sspa.sspa_SVD to yield the N-by-P pathway activity-score matrix.

## Related tools

- **sspa** (Python package whose README documents the z-score standardisation step and provides the pathway scoring (sspa_kpca, sspa_SVD) that consumes the scaled matrices) — github.com/cwieder/py-ssPA
- **PathIntegrate** (Applies unit-variance standardisation automatically to each input omics matrix before pathway transformation) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (sspa methods expose fit/transform/fit_transform for compatibility with SKlearn Pipeline and GridSearchCV, so scaled matrices can flow through standard ML workflows)
- **Python** (Implementation language for the pandas-based log-transform, median imputation and column-wise z-score scaling shown in the README)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- After scaling, every molecule column has mean ≈ 0 and standard deviation ≈ 1 (verify with matrix.mean(axis=0) and matrix.std(axis=0))
- The scaled matrix retains one row per retained sample and one column per mapped molecule identifier matching the pathway database (ChEBI/UniProt/ENSEMBL or KEGG IDs)
- Downstream sspa_kpca.fit_transform produces an N × P matrix with one row per input sample and finite (non-missing) pathway scores for every pathway containing observed molecules
- Log-transform was applied before z-scoring, consistent with the README's processed-data recipe for the Su et al. 2020 COVID-19 metabolomics dataset
- No raw (unscaled) values remain: each column's values span roughly ±3–4 SD rather than the original dynamic range of raw intensities

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals, so samples from individuals without data on all omics must be discarded and cannot be scaled jointly
- Un-annotated molecules that fail identifier mapping to the pathway database cannot be incorporated, causing information loss before standardisation and pathway scoring
- The reliance on current pathway annotation knowledge is a widely accepted limitation of pathway-based methods, including this scaling-plus-ssPA pipeline
- Metabolite columns with >50% NA/0/' ' values are filtered out (README example), so very sparse features never reach the standardisation step

## Evidence

- [methods] Standardise each molecular abundance matrix to unit variance (z-score scaling) as PathIntegrate automatically standardises input data, with log-transformation recommended beforehand.: [evidence span withheld — anchor status: not_found]
- [readme] README z-score recipe: [evidence span withheld — anchor status: external_doc]
- [intro] common-scale rationale: "PathIntegrate substantially contributes to addressing these with the pathway-transformation step, where disparate omics datasets are brought to a common scale"
- [discussion] missing-row constraint: "PathIntegrate requires all input omics to be measured on the same individuals. This means samples from individuals without data on all omics will have to be discarded, as PathIntegrate currently does"
- [readme] README missingness filter: [evidence span withheld — anchor status: external_doc]
