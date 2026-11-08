---
name: multi-omics-pathway-transformation
description: Use when you have two or more molecular abundance matrices (e.g. log-transformed
  metabolite and protein values with ChEBI/UniProt/Gene Symbol IDs) measured on the
  same individuals and need interpretable, pathway-level features for predictive multi-omics
  modelling.
when_to_use_negative:
- 'Samples from individuals without data on all omics: PathIntegrate requires all
  input omics to be measured on the same individuals, and such samples must be discarded'
- You need to incorporate un-annotated molecules (no ChEBI/UniProt/ENSEMBL mapping
  into pathway databases) — pathway-based models lose this information
- Only one omics layer is available with no need for multi-omics pathway-level integration
  (conventional ORA/GSEA on a single dataset may suffice)
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: sspa
  role: Provides single-sample pathway analysis methods (sspa_SVD/PLAGE, sspa_KPCA,
    ssClustPA), pathway database downloads (KEGG, Reactome, PathBank) and multi-omics
    identifier harmonisation (ChEBI/UniProt/Gene Symbol) used to generate the pathway
    scores
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Consumes the pathway-transformed multi-omics dataset for Single-View (SKlearn-compatible
    predictive model) or Multi-View (MB-PLS) integration, automatic standardisation
    and ranking of multi-omics pathways
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: ssPA transformations expose fit/transform/fit_transform for compatibility
    with SKlearn functions such as Pipeline and GridSearchCV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:354745765de93d08607a2172d295dfd7557ca97ca59ed3a37b0f3b362f780f11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# multi-omics-pathway-transformation

## Summary

Transform molecular-level metabolomics/proteomics abundance matrices into a unified pathway-score matrix using single-sample pathway analysis (ssPA), the core pathway-transformation step of the PathIntegrate framework for multi-omics integration.

## When to use

Use when you have two or more molecular abundance matrices (e.g. log-transformed metabolite and protein values with ChEBI/UniProt/Gene Symbol IDs) measured on the same individuals and need interpretable, pathway-level features for predictive multi-omics modelling.

## When NOT to use

- Samples from individuals without data on all omics: PathIntegrate requires all input omics to be measured on the same individuals, and such samples must be discarded
- You need to incorporate un-annotated molecules (no ChEBI/UniProt/ENSEMBL mapping into pathway databases) — pathway-based models lose this information
- Only one omics layer is available with no need for multi-omics pathway-level integration (conventional ORA/GSEA on a single dataset may suffice)

## Inputs

- Molecular-level abundance matrices (pandas DataFrames, samples as rows, molecule IDs as columns) for each omics layer, e.g. metabolomics with ChEBI IDs and proteomics with UniProt/Gene Symbol IDs, log-transformed and imputed
- Multi-omics pathway definitions from sspa (Reactome v78 via sspa.process_reactome or KEGG v98 via sspa.process_kegg, or a custom GMT file via sspa.process_gmt)
- Minimum pathway-size threshold (min_entity, e.g. 2 measured molecules per pathway)

## Outputs

- A multi-omics pathway-transformed dataset (pathway-score matrix) with one column per retained pathway, e.g. 732 multi-omics pathway scores for the Su et al. COVID-19 metabolomics + proteomics data
- Per-omics pathway score matrices compatible with SKlearn pipelines for downstream Single-View (e.g. RandomForestClassifier + RFECV) or Multi-View (MB-PLS) modelling

## How to apply

Load Reactome (or KEGG) multi-omics pathway definitions via the sspa package, e.g. sspa.process_reactome('Homo sapiens', omics_type='multiomics', identifiers=['chebi','uniprot','gene_symbol']) or sspa.process_kegg('hsa', omics_type='multiomics'). Each molecule in the abundance matrices must map to a pathway database identifier (ChEBI, UniProt, or ENSEMBL; KEGG compound/gene for KEGG). Apply an ssPA scoring method such as the SVD-based PLAGE (sspa.sspa_SVD) or kPCA (sspa.sspa_KPCA) to each omics layer, retaining only pathways with at least 2 measured molecules (min_entity=2); the article reports 732 multi-omics pathways retained at this threshold for the Su et al. COVID-19 data. Concatenate the resulting pathway-score columns from all omics layers into one multi-omics pathway-transformed dataset, which brings the disparate omics datasets to a common scale and serves as input to a Single-View or Multi-View model. ssPA methods expose fit/transform/fit_transform for SKlearn Pipeline and GridSearchCV compatibility; fit the transformation on training data only and transform the held-out test set with the fitted model.

## Related tools

- **sspa** (Provides single-sample pathway analysis methods (sspa_SVD/PLAGE, sspa_KPCA, ssClustPA), pathway database downloads (KEGG, Reactome, PathBank) and multi-omics identifier harmonisation (ChEBI/UniProt/Gene Symbol) used to generate the pathway scores) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Consumes the pathway-transformed multi-omics dataset for Single-View (SKlearn-compatible predictive model) or Multi-View (MB-PLS) integration, automatic standardisation and ranking of multi-omics pathways) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (ssPA transformations expose fit/transform/fit_transform for compatibility with SKlearn functions such as Pipeline and GridSearchCV)

## Examples

```
reactome_mo = sspa.process_reactome('Homo sapiens', omics_type='multiomics', identifiers=['chebi','uniprot','gene_symbol']); pathway_scores = sspa.sspa_SVD(reactome_mo, min_entity=2).fit_transform(log2_protein_metab_matrix)
```

## Evaluation signals

- Every molecule ID in the input matrices must map to a valid pathway-database identifier (ChEBI, UniProt, or ENSEMBL) before scoring
- Pathways retained in the transformed dataset satisfy the min_entity threshold (e.g. ≥2 measured molecules); verify the count of retained pathways matches expectations (732 multi-omics pathways for the COVID-19 case study)
- Pathway-score matrix has one column per retained pathway and rows aligned with the input samples; no samples from individuals lacking data on all omics
- Transformation fit only on training data and applied via transform() to test data (no test-set leakage through the ssPA fit)
- Downstream Single-View model on the transformed data reproduces reported performance (e.g. AUC ~0.95 vs 0.98 for the molecular-level concatenated baseline on the COVID-19 severity task)

## Limitations

- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics
- Reliance on current pathway annotation knowledge is a widely accepted limitation; pathway definitions and sizes vary between databases
- Un-annotated molecules cannot currently be incorporated into pathway-based models, causing information loss
- It is a challenge to compare molecular vs pathway-level models because it is unknown how many molecules in a pathway are differentially abundant at any one time
- sspa methods have fit/transform/fit_transform, but for sspa_ssClustPA, sspa_SVD, and sspa_KPCA specifically the model must be fit on training data and test data transformed with the fitted model

## Evidence

- [abstract] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [readme] sspa README: kPCA scoring example: [evidence span withheld — anchor status: external_doc]
- [readme] sspa README: Reactome multi-omics pathway download: [evidence span withheld — anchor status: external_doc]
- [readme] sspa README: SKlearn compatibility of ssPA methods: [evidence span withheld — anchor status: not_found]
- [methods] Pathways downloaded using sspa package: "Pathways can be downloaded using the sspa package (github.com/cwieder/py-ssPA)"
- [discussion] Discussion: pathway coverage limitation: [evidence span withheld — anchor status: not_found]
- [discussion] Discussion: same-individual requirement: "PathIntegrate requires all input omics to be measured on the same individuals. This means samples from individuals without data on all omics will have to be discarded, as PathIntegrate currently does"
