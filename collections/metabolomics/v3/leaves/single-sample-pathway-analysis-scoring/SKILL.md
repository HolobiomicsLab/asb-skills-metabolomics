---
name: single-sample-pathway-analysis-scoring
description: Use when molecular-level omics matrices need a common, interpretable
  scale for supervised integration — e.g. before fitting MB-PLS or a sklearn classifier
  on multi-omics COPDgene or COVID-19 data — to exploit coordinated pathway-level
  signals in low signal-to-noise scenarios.
when_to_use_negative:
- Input is already a pathway-level score matrix — re-scoring would be redundant; the
  skill applies to molecular-level abundance data
- PathIntegrate requires all input omics to be measured on the same individuals —
  samples with missing entire omics rows must be excluded before scoring, as the tool
  does not support rows of missing data
- Molecules lacking valid ChEBI/UniProt/ENSEMBL annotations cannot be included in
  pathway scoring, so un-annotated molecule lists cause information loss rather than
  pathway scores
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0154
tools:
- name: sspa
  role: Provides sspa_kpca and sspa_SVD (PLAGE) scoring methods plus pathway database
    loaders (process_reactome, process_kegg, process_pathbank, process_gmt) used to
    transform molecular matrices into pathway scores
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Consumes the pathway score matrices as input for Single-View (sklearn-compatible
    classifier) and Multi-View (MB-PLS) supervised integration models
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: ssPA transformers expose fit/transform/fit_transform for use inside sklearn
    Pipeline and GridSearchCV for parameter tuning and cross-validation
  repo: https://github.com/cwieder/py-ssPA
- name: Python
  role: Runtime environment in which the sspa transformation and downstream PathIntegrate
    modelling are executed
  repo: https://github.com/cwieder/PathIntegrate
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  - task_005
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:8c36afe1fb7326dfc034e2cb964dbe078bf6535225d047e37082f8baa8dc086a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  - sspa (py-ssPA)
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# single-sample-pathway-analysis-scoring

## Summary

Transforms an N-by-M molecular abundance matrix (metabolites, proteins, or transcripts with ChEBI/UniProt/ENSEMBL or KEGG IDs) into an N-by-P pathway activity-score matrix using kPCA or SVD/PLAGE scoring from the sspa Python package, the pathway-transformation step that underpins PathIntegrate Single-View and Multi-View models.

## When to use

Use when molecular-level omics matrices need a common, interpretable scale for supervised integration — e.g. before fitting MB-PLS or a sklearn classifier on multi-omics COPDgene or COVID-19 data — to exploit coordinated pathway-level signals in low signal-to-noise scenarios.

## When NOT to use

- Input is already a pathway-level score matrix — re-scoring would be redundant; the skill applies to molecular-level abundance data
- PathIntegrate requires all input omics to be measured on the same individuals — samples with missing entire omics rows must be excluded before scoring, as the tool does not support rows of missing data
- Molecules lacking valid ChEBI/UniProt/ENSEMBL annotations cannot be included in pathway scoring, so un-annotated molecule lists cause information loss rather than pathway scores

## Inputs

- Molecular abundance matrix as a pandas DataFrame: samples as rows, molecule identifiers (ChEBI/UniProt/ENSEMBL for Reactome, KEGG compound/gene IDs for KEGG) as columns
- Pathway DataFrame from sspa (pathways as rows, molecules as columns), from KEGG, Reactome, PathBank, or a custom GMT file (.gmt or .csv)
- Optional omics-type specification for multi-omics pathways (e.g. omics_type='multiomics', identifiers=['chebi','uniprot','gene_symbol'])
- min_entity threshold (e.g. min_entity=2) controlling minimum molecules measured per pathway

## Outputs

- N-by-P pathway activity-score matrix with sample IDs as index and pathway IDs as columns, written e.g. to pathway_scores.csv
- Fitted ssPA transformer object (kpca_scores or SVD scores) compatible with sklearn Pipeline and GridSearchCV

## How to apply

Load each omics abundance matrix as a pandas DataFrame with samples as rows and molecule identifiers (ChEBI, UniProt, or ENSEMBL for Reactome; KEGG compound/gene IDs for KEGG) as columns, ensuring identifiers match the chosen pathway database; log-transform and z-score each molecular matrix to unit variance as PathIntegrate automatically standardises input data. Fetch pathway definitions with sspa.process_reactome('Homo sapiens') or sspa.process_kegg('hsa'), accepting pathways drawn from a single omics or a multi-omics combination (omics_type='multiomics' with identifiers like ['chebi','uniprot','gene_symbol']). Transform each matrix with sspa.sspa_kpca(pathways, min_entity=2).fit_transform(molecule_matrix) or the SVD-based sspa.sspa_SVD PLAGE method, retaining only pathways with at least 2 measured molecules (the article reports 732 multi-omics Reactome pathways retained at this threshold for the Su et al. COVID-19 data). The ssPA methods expose fit(), transform(), and fit_transform() so the model can be fit on training data and applied to test data within sklearn Pipeline and GridSearchCV workflows. Validate that the output pathway-score matrix has one row per input sample, one column per scored pathway, and finite values for all pathways containing observed molecules; this transformed matrix is then used to benchmark univariate MWU signal detection or as input to PathIntegrate Single-View/Multi-View models.

## Related tools

- **sspa** (Provides sspa_kpca and sspa_SVD (PLAGE) scoring methods plus pathway database loaders (process_reactome, process_kegg, process_pathbank, process_gmt) used to transform molecular matrices into pathway scores) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Consumes the pathway score matrices as input for Single-View (sklearn-compatible classifier) and Multi-View (MB-PLS) supervised integration models) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (ssPA transformers expose fit/transform/fit_transform for use inside sklearn Pipeline and GridSearchCV for parameter tuning and cross-validation) — https://github.com/cwieder/py-ssPA
- **Python** (Runtime environment in which the sspa transformation and downstream PathIntegrate modelling are executed) — https://github.com/cwieder/PathIntegrate

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- Output matrix must have one row per input sample and one column per scored pathway, matching the N-by-P shape described in the article
- All pathway score values must be finite (non-missing) for pathways containing molecules observed in the input data
- Pathways with fewer than 2 measured molecules (min_entity=2) must be excluded from the score matrix
- Molecular input must be standardised to unit variance (z-score scaling) before transformation — PathIntegrate performs automatic standardisation
- When scoring a training/test split, the fitted transformer must be applied via transform() to the test set rather than refit, per the sklearn-compatible API

## Limitations

- Pathway coverage — the proportion of molecules in a pathway observable in the data — is an inherent bottleneck, particularly for metabolomics
- Reliance on current pathway annotation knowledge is a widely accepted limitation; un-annotated molecules cannot currently be incorporated, causing information loss
- PathIntegrate requires all input omics to be measured on the same individuals, so samples missing an entire omics layer must be discarded
- Molecules must carry valid identifiers (ChEBI, UniProt, or ENSEMBL for Reactome; KEGG compound/gene for KEGG) matching the chosen pathway database

## Evidence

- [methods] ssPA takes an N-by-M matrix of molecules as input and combines the abundance values of molecules in predefined pathways to yield an N-by-P pathway-level matrix: [evidence span withheld — anchor status: not_found]
- [intro] single-sample pathway analysis approaches transform molecular-level abundance data matrices into pathway-level matrices: "PathIntegrate employs single-sample pathway analysis approaches (ssPA) (Fig 1), which transform molecular-level abundance data matrices into pathway-level matrices"
- [readme] Generate pathway scores using kPCA method via sspa.sspa_kpca with min_entity=2: [evidence span withheld — anchor status: external_doc]
- [readme] All ssPA methods now have fit(), transform() and fit_transform() for SciKitLearn compatibility: [evidence span withheld — anchor status: external_doc]
- [readme] download multi-omics pathways from Reactome with specified identifier types: [evidence span withheld — anchor status: external_doc]
- [discussion] pathway coverage is an inherent bottleneck of pathway-based analyses: [evidence span withheld — anchor status: not_found]
