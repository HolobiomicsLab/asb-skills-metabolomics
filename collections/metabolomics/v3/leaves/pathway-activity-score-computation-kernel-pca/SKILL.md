---
name: pathway-activity-score-computation-kernel-pca
description: Use when you have one or more standardised, log-transformed molecular
  abundance matrices (metabolomics with ChEBI/KEGG IDs, proteomics or RNA-seq with
  UniProt/ENSEMBL or gene symbols) and need per-sample pathway-level feature scores
  to feed supervised multi-omics integration models.
when_to_use_negative:
- Input matrix has not been log-transformed and z-score standardised per molecule;
  kPCA scores are unreliable on raw or unnormalised abundance values
- Molecule identifiers do not match the pathway database (e.g. metabolite names against
  Reactome ChEBI IDs without prior sspa.identifier_conversion and sspa.map_identifiers
  harmonisation)
- The goal is conventional differential-molecule pathway testing (ORA or GSEA), not
  per-sample pathway scoring for supervised prediction
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0602
tools:
- name: sspa
  role: Provides sspa_kpca (kernel PCA single-sample pathway analysis) fit_transform
    and the pathway database loaders (process_reactome, process_kegg, process_gmt)
    used to compute pathway activity scores
  repo: github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Consumes the kPCA-derived pathway score matrices as pathway-transformed input
    for Single-View and Multi-View (MB-PLS) supervised integration models
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Provides the Pipeline and GridSearchCV machinery with which sspa_kpca's fit/transform/fit_transform
    interface is compatible for model optimisation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:b7f132d63e79c2805e99c553658963ff0e279e59f1db2091f71fa0da226bf867
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pathway-activity-score-computation-kernel-pca

## Summary

Computes per-sample pathway activity-score matrices from standardised molecular abundance data using the kernel PCA (sspa_kpca) implementation in the sspa package, converting an N-by-M molecule matrix into an N-by-P pathway matrix for PathIntegrate Single-View and Multi-View models.

## When to use

Use when you have one or more standardised, log-transformed molecular abundance matrices (metabolomics with ChEBI/KEGG IDs, proteomics or RNA-seq with UniProt/ENSEMBL or gene symbols) and need per-sample pathway-level feature scores to feed supervised multi-omics integration models.

## When NOT to use

- Input matrix has not been log-transformed and z-score standardised per molecule; kPCA scores are unreliable on raw or unnormalised abundance values
- Molecule identifiers do not match the pathway database (e.g. metabolite names against Reactome ChEBI IDs without prior sspa.identifier_conversion and sspa.map_identifiers harmonisation)
- The goal is conventional differential-molecule pathway testing (ORA or GSEA), not per-sample pathway scoring for supervised prediction

## Inputs

- Processed and standardised molecular abundance matrix per omics (samples x molecules, pandas DataFrame) with IDs matching Reactome (ChEBI, UniProt, ENSEMBL) or KEGG compound identifiers
- Pathway database as a pandas DataFrame from sspa.process_reactome or sspa.process_kegg (pathways x molecules), including multi-omics variants (omics_type='multiomics')
- Minimum pathway size parameter (min_entity, e.g. 2 molecules)

## Outputs

- N-by-P pathway activity-score matrix (pathway_scores.csv) with one row per sample and one column per scored pathway
- Optional alternative pathway scores from PLAGE (sspa.sspa_SVD) for benchmarking

## How to apply

Load each omics molecular abundance matrix as a pandas DataFrame with samples as rows and molecule IDs as columns, ensuring the column identifiers (ChEBI, UniProt, ENSEMBL, or KEGG compounds) match the chosen pathway database (Reactome via sspa.process_reactome or KEGG via sspa.process_kegg). Standardise every molecule to unit variance (z-score scaling) and apply a log2 transform beforehand, since kPCA is sensitive to scale. For each pathway, the sspa.sspa_kpca fit_transform method fits a kernel PCA embedding of the molecules belonging to that pathway (respecting min_entity, e.g. minimum 2 molecules) and takes the resulting embedding score as the sample's pathway activity value. The output is an N-by-P DataFrame indexed by sample ID with pathway IDs as columns, written to pathway_scores.csv for downstream Single-View (sklearn model) or Multi-View (MB-PLS) integration. Validate that every sample row is present and that all retained pathways contain molecules observed in the data with finite (non-missing) score values.

## Related tools

- **sspa** (Provides sspa_kpca (kernel PCA single-sample pathway analysis) fit_transform and the pathway database loaders (process_reactome, process_kegg, process_gmt) used to compute pathway activity scores) — github.com/cwieder/py-ssPA
- **PathIntegrate** (Consumes the kPCA-derived pathway score matrices as pathway-transformed input for Single-View and Multi-View (MB-PLS) supervised integration models) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Provides the Pipeline and GridSearchCV machinery with which sspa_kpca's fit/transform/fit_transform interface is compatible for model optimisation)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- Output pathway score matrix has exactly one row per input sample and one column per pathway in the loaded database that contains molecules observed in the data
- All pathway score values are finite (non-missing, non-NaN) after fit_transform
- Column identifiers of the input molecular matrix match the identifier scheme of the loaded pathway database (ChEBI/UniProt/ENSEMBL for Reactome, KEGG compounds for KEGG)
- Input molecular values are z-score standardised (mean 0, std 1) per molecule, typically after log2 transformation
- Each scored pathway retains at least min_entity molecules present in the data (e.g. minimum 2 for the COVID-19 example)

## Limitations

- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, so some pathways may have few or no scored molecules
- PathIntegrate does not support entire rows of missing data; samples from individuals lacking data on all omics must be discarded
- Un-annotated molecules cannot currently be incorporated into pathway-based models, causing information loss
- Reliance on current pathway annotation knowledge in databases such as Reactome v78 or KEGG v98 is a widely accepted limitation

## Evidence

- [methods] ssPA takes an N-by-M matrix of molecules as input and combines the abundance values of molecules in predefined pathways to yield an N-by-P pathway-level matrix: [evidence span withheld — anchor status: not_found]
- [readme] Generate pathway scores using kPCA method: "Generate pathway scores using kPCA method

kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])"
- [readme] Standardise the data (metabolite values) using z-score (mean=0, std=1): [evidence span withheld — anchor status: external_doc]
- [readme] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [methods] Load each omics molecular abundance matrix as a pandas DataFrame with samples as rows and molecule IDs: [evidence span withheld — anchor status: not_found]
