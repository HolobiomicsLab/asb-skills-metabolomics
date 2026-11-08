---
name: pathway-database-comparison-kegg-reactome
description: Use when choosing between KEGG and Reactome as the pathway source for
  pathway-transformed multi-omics models, or when quantifying database-specific pathway
  sizes and molecule-to-pathway identifier coverage.
when_to_use_negative:
- The pathway database has already been fixed by the study design and no comparison
  is required
- Molecule identifiers cannot be mapped to either database's identifier scheme (KEGG
  compound IDs vs ChEBI/UniProt), making one database unusable for the data
- The goal is differential pathway analysis on a single omics dataset rather than
  evaluating database choice for predictive multi-omics modelling
tools:
- name: sspa (py-ssPA)
  role: Downloads KEGG and Reactome pathway databases and performs the single-sample
    pathway analysis (ssPA) transformations used per database
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Runs the pathway-based Multi-View (MB-PLS) and Single-View (PLS-DA) models
    on each database's pathway scores for the AUROC comparison
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Baseline model benchmarked at the molecular level and on pathway scores ('DIABLO
    pathway') for each pathway database
- name: SciKitLearn
  role: Provides the PLS-DA classifier and Pipeline/GridSearchCV integration for cross-validated
    tuning of latent variables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:abed81dbfbbfcb95b169d7105ab2983cd616184a16db5da85df7baf2d7386e57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa (py-ssPA)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pathway-database-comparison-kegg-reactome

## Summary

Benchmarks KEGG vs Reactome as the pathway reference for pathway-based multi-omics modelling: both databases are downloaded via the sspa package, used to transform the same molecular data into pathway scores, and compared on downstream predictive performance (AUROC) and pathway size/coverage.

## When to use

Use when choosing between KEGG and Reactome as the pathway source for pathway-transformed multi-omics models, or when quantifying database-specific pathway sizes and molecule-to-pathway identifier coverage.

## When NOT to use

- The pathway database has already been fixed by the study design and no comparison is required
- Molecule identifiers cannot be mapped to either database's identifier scheme (KEGG compound IDs vs ChEBI/UniProt), making one database unusable for the data
- The goal is differential pathway analysis on a single omics dataset rather than evaluating database choice for predictive multi-omics modelling

## Inputs

- Molecular-level multi-omics abundance DataFrames (samples as rows, molecule IDs as columns), e.g. COPDgene metabolomics and proteomics
- KEGG human pathway matrix (KEGG compound and gene identifiers)
- Reactome human pathway matrix (ChEBI, UniProt, and Gene Symbol identifiers)
- Outcome labels (e.g. COPD status)

## Outputs

- Pathway-level score matrices generated separately for KEGG and Reactome
- Pathway size distributions (log10 pathway size violin plots) per database
- Identifier mapping coverage statistics (percentage of molecules with valid ChEBI/UniProt/ENSEMBL IDs mapping to pathways)
- Held-out test AUROC values per model, effect size, and pathway database (e.g. fig4A_auroc_results.csv)

## How to apply

Download both pathway databases through the sspa package (sspa.process_reactome for Reactome v78, sspa.process_kegg for KEGG v98), noting that KEGG pathways use KEGG compound/gene identifiers while Reactome uses ChEBI, UniProt, and Gene Symbol identifiers. Map the molecular-level metabolomics/proteomics columns onto each database and compute per-database pathway coverage, since pathway coverage is an inherent bottleneck especially for metabolomics. Transform the same molecular datasets into pathway-level scores independently for KEGG and Reactome using ssPA methods (e.g. sspa_kpca) inside the PathIntegrate pipeline. Fit the identical models (PathIntegrate Multi-View MB-PLS, Single-View PLS-DA, DIABLO baselines) on each database's pathway scores under the same 5-times repeated nested 5-fold cross-validation. Compare held-out test AUROC per database across effect sizes, expecting KEGG to be marginally better than Reactome at larger effect sizes.

## Related tools

- **sspa (py-ssPA)** (Downloads KEGG and Reactome pathway databases and performs the single-sample pathway analysis (ssPA) transformations used per database) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Runs the pathway-based Multi-View (MB-PLS) and Single-View (PLS-DA) models on each database's pathway scores for the AUROC comparison) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Baseline model benchmarked at the molecular level and on pathway scores ('DIABLO pathway') for each pathway database)
- **SciKitLearn** (Provides the PLS-DA classifier and Pipeline/GridSearchCV integration for cross-validated tuning of latent variables)

## Examples

```
reactome_pathways = sspa.process_reactome(organism="Homo sapiens"); kegg_pathways = sspa.process_kegg(organism="hsa")
```

## Evaluation signals

- Pathway downloads are versioned and reproducible (Reactome v78, KEGG v98 as pre-loaded in sspa); pathway size distributions should be reported as log10 violin plots per database
- Identifier-mapping coverage is quantified: percentage of molecules with a valid identifier (ChEBI, UniProt, or ENSEMBL) mapping to pathways, per database
- Identical models and tuning (5-times repeated nested 5-fold CV for number of latent variables) are applied to each database's pathway scores, so AUROC differences reflect the database, not the pipeline
- Reproduced pattern: pathway-based models outperform molecular-level DIABLO across all effect sizes, with KEGG marginally better than Reactome at larger effect sizes
- Per-database pathway transformations are computed independently (no mixing of KEGG and Reactome pathway IDs in one matrix)

## Limitations

- Pathway definitions and sizes vary between databases, making molecular-vs-pathway and cross-database comparisons inherently difficult
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck of pathway-based analyses, particularly for metabolomics
- Results depend on current pathway annotation knowledge, a widely accepted limitation of pathway-based methods

## Evidence

- [methods] Support for multiple pathway databases, including KEGG and Reactome: "Support for multiple pathway databases, including KEGG and Reactome"
- [readme] Note: KEGG pathways use KEGG compound IDs, Reactome and Pathbank pathways use ChEBI and UniProt: [evidence span withheld — anchor status: external_doc]
- [supplementary] Violin plots showing log10 pathway size for KEGG and Reactome human databases: "Violin plots showing log10 pathway size for KEGG and Reactome human databases"
- [discussion] it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant: "it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant"
- [supplementary] Percentage of molecules with a valid identifier (ChEBI, UniProt, or ENSEMBL) in single omics mapping to Reactome human pathways: "Percentage of molecules with a valid identifier (ChEBI, UniProt, or ENSEMBL) in single omics mapping to Reactome human pathways"
