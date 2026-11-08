---
name: permutation-based-significance-testing
description: Use when a pathway-level importance statistic (MB-VIP, VIP, RGCCA loading
  relevance, bootstrap non-zero-loading count) needs a p-value, and no closed-form
  null exists because the statistic comes from a fitted multivariate model (MB-PLS,
  PLS, DIABLO).
when_to_use_negative:
- The goal is differential-abundance testing of individual molecules, where standard
  statistical tests (e.g. t-tests used in sspa ORA) provide the null directly and
  permutation of model statistics is unnecessary.
- Samples are not measured across all omics views — PathIntegrate requires complete
  multi-omics data, so individuals missing an omics layer must be excluded before
  model fitting and permutation.
- The analysis targets conventional pathway analysis (ORA, GSEA, MultiGSEA, IMPaLA)
  rather than pathway importance statistics from a predictive model; the paper notes
  no direct comparison to these approaches was performed.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Provides the MB-PLS / Single-View models whose MB-VIP and VIP pathway statistics
    are permuted and tested; also the DIABLO benchmark interface
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Model fitting and evaluation machinery that the permutation loop refits on
    each shuffled label vector; ssPA transformers and PathIntegrate are built to be
    SciKitLearn-compatible
- name: sspa
  role: Supplies the kPCA single-sample pathway transformation and Reactome pathway
    retrieval that produce the pathway-level inputs whose importance statistics are
    tested
  repo: https://github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:41634d610eca97b4a4e3e5ccf680370e19d26a8c0b970619a72330cb0a69fdd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# permutation-based-significance-testing

## Summary

Assess significance of pathway importance statistics (MB-VIP, VIP, DIABLO loading relevance) by building empirical null distributions through repeated label permutation and model refitting, then applying Benjamini-Hochberg FDR correction. This is how PathIntegrate attaches p-values to multi-omics pathway rankings.

## When to use

Use when a pathway-level importance statistic (MB-VIP, VIP, RGCCA loading relevance, bootstrap non-zero-loading count) needs a p-value, and no closed-form null exists because the statistic comes from a fitted multivariate model (MB-PLS, PLS, DIABLO).

## When NOT to use

- The goal is differential-abundance testing of individual molecules, where standard statistical tests (e.g. t-tests used in sspa ORA) provide the null directly and permutation of model statistics is unnecessary.
- Samples are not measured across all omics views — PathIntegrate requires complete multi-omics data, so individuals missing an omics layer must be excluded before model fitting and permutation.
- The analysis targets conventional pathway analysis (ORA, GSEA, MultiGSEA, IMPaLA) rather than pathway importance statistics from a predictive model; the paper notes no direct comparison to these approaches was performed.

## Inputs

- Fitted PathIntegrate Multi-View (MB-PLS) or Single-View model with pathway importance scores (MB-VIP/VIP)
- Fitted DIABLO / sparse DIABLO model with RGCCA component loadings
- Pathway-transformed multi-omics matrices (ssPA kPCA pathway scores, e.g. Reactome pathways from the sspa package)
- Outcome labels (e.g. COPD status) subject to permutation
- Number of permutations (10,000) and FDR correction setting

## Outputs

- Permutation null distributions of the pathway importance statistic (10,000 values per statistic)
- Empirical permutation p-values per pathway per omics view
- Benjamini-Hochberg FDR-corrected significance table (e.g. mbpls_pathway_importance.csv with pathway name, omics view, MB-VIP score, p-value)
- Pathway detection confusion matrices and confusion-matrix AUCs for benchmarking (pathway_detection_auc.csv)

## How to apply

First fit the pathway-based model on the pathway-transformed data (e.g. ssPA kPCA scores of Reactome pathways) and compute the observed importance statistic per pathway per omics view, such as block-wise MB-VIP for the Multi-View MB-PLS model or RGCCA component-loading relevance for DIABLO. Generate a null distribution by permuting the outcome labels (10,000 permutations in the semi-synthetic pathway-detection benchmark), refitting the model and recomputing the statistic each time; for the sparse L1-norm DIABLO variant the detection statistic is a bootstrap non-zero-loading statistic. Convert the observed statistic to an empirical permutation p-value against the null, then apply Benjamini-Hochberg FDR correction across pathways. Interpret per-omics-view results: in the COPDgene Multi-View model carnitine metabolism was significant in the metabolomics layer (p = 0.003) but not transcriptomics (p = 0.55), and surfactant metabolism was significant in proteomics (p = 0.002), showing how the permutation test attributes pathway significance to specific omics layers.

## Related tools

- **PathIntegrate** (Provides the MB-PLS / Single-View models whose MB-VIP and VIP pathway statistics are permuted and tested; also the DIABLO benchmark interface) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Model fitting and evaluation machinery that the permutation loop refits on each shuffled label vector; ssPA transformers and PathIntegrate are built to be SciKitLearn-compatible)
- **sspa** (Supplies the kPCA single-sample pathway transformation and Reactome pathway retrieval that produce the pathway-level inputs whose importance statistics are tested) — https://github.com/cwieder/py-ssPA

## Evaluation signals

- The permutation null must contain 10,000 refitted statistic values (10,000 permutations) before p-value computation.
- Reproduced COPDgene p-values should match reported values: carnitine metabolism metabolomics p = 0.003 and transcriptomics p = 0.55; surfactant metabolism proteomics p = 0.002.
- All permutation p-values must lie in [0, 1] and BH-FDR-corrected q-values must not decrease relative to raw p-values.
- In the semi-synthetic benchmark, confusion-matrix AUCs built from the permutation-tested statistics should reach ≥ 0.90 for the target pathway even at effect size 0.25.
- Permuting labels should drive p-values toward uniformity — pathways that were significant under true labels should lose significance under shuffled labels.

## Limitations

- The empirical null is conditional on the fitted model: any change to latent-variable count, pathway database (Reactome vs KEGG), or pathway transformation invalidates the stored null distribution and requires re-permutation.
- Permutation testing at 10,000 refits is computationally expensive for high-dimensional pathway matrices and repeated nested cross-validation settings.
- Significance depends on pathway coverage and annotation; metabolites lacking ChEBI/Reactome annotation cannot contribute to the tested pathway statistics.
- Reported p-values are specific to COPDgene Phase-2 data and semi-synthetic benchmarks and are not guaranteed to transfer to other cohorts.

## Evidence

- [other] Permutation null and FDR correction in the detection benchmark: [evidence span withheld — anchor status: not_found]
- [other] Permutation testing of MB-VIP with layer-specific p-values: [evidence span withheld — anchor status: not_found]
- [abstract] Rationale for pathway-level signal detection that the permutation test evaluates: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [readme] SciKitLearn compatibility underpinning the refit-per-permutation loop: [evidence span withheld — anchor status: external_doc]
- [other] Expected detection performance validated via permutation-based statistics: [evidence span withheld — anchor status: not_found]
