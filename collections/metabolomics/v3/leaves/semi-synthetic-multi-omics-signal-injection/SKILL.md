---
name: semi-synthetic-multi-omics-signal-injection
description: Use when no real benchmarking dataset with validated molecular- and pathway-level
  signals exists, and you need controlled multi-omics ground truth across effect sizes
  to compare molecular-level vs pathway-level predictive models.
when_to_use_negative:
- The analysis goal is comparison against conventional pathway analysis approaches
  (ORA, GSEA, MultiGSEA, IMPaLA) — the authors state a full comparison was beyond
  the scope of the work
- Samples lack measurements for all omics views — PathIntegrate requires all input
  omics on the same individuals, so rows with missing entire views must be discarded
  and injection would not fix this
- Most molecules of interest are un-annotated to pathway databases — pathway coverage/annotation
  bottlenecks (especially for metabolomics) mean injected pathway signals cannot be
  represented
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: sspa (py-ssPA)
  role: Downloads KEGG/Reactome multi-omics pathway databases and provides preprocessing
    and ssPA transformations used to build and validate the semi-synthetic pathway
    signals
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Fits Multi-View (MB-PLS) and Single-View (scikit-learn PLS-DA) models on the
    semi-synthetic pathway-transformed data to obtain benchmark AUROC
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Molecular-level and pathway-level multi-omics baseline model benchmarked against
    PathIntegrate on the semi-synthetic data
- name: SciKitLearn
  role: Provides the predictive models, Pipeline and GridSearchCV used for ssPA-transformed
    data and nested cross-validated tuning of latent variables
- name: PathIntegrate_scripts
  role: Benchmarking scripts (developed for a PBS HPC cluster) used to run the semi-synthetic
    COPDgene experiments
  repo: https://github.com/cwieder/PathIntegrate_scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:9a5ba6a34d5c792958abb14c2cd61ca5f6ccf2515f19c9600a096cf7d711760a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - DIABLO
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - sspa (py-ssPA)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# semi-synthetic multi-omics signal injection

## Summary

Generates semi-synthetic multi-omics benchmark data (e.g. COPDgene metabolomics + proteomics) by injecting controlled differential abundance signals into known pathways at prescribed effect sizes, enabling validation that pathway-level models (PathIntegrate Multi-View/Single-View) outperform molecular-level baselines (DIABLO) by held-out test AUROC.

## When to use

Use when no real benchmarking dataset with validated molecular- and pathway-level signals exists, and you need controlled multi-omics ground truth across effect sizes to compare molecular-level vs pathway-level predictive models.

## When NOT to use

- The analysis goal is comparison against conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) — the authors state a full comparison was beyond the scope of the work
- Samples lack measurements for all omics views — PathIntegrate requires all input omics on the same individuals, so rows with missing entire views must be discarded and injection would not fix this
- Most molecules of interest are un-annotated to pathway databases — pathway coverage/annotation bottlenecks (especially for metabolomics) mean injected pathway signals cannot be represented

## Inputs

- Real multi-omics abundance DataFrames (samples x molecules), e.g. COPDgene metabolomics and proteomics with molecule identifiers as columns
- Pathway databases (KEGG, Reactome) loaded or downloaded via the sspa package as molecule-to-pathway membership matrices
- Outcome labels (e.g. COPD status or gender) used to derive fold changes for signal injection
- List of prescribed effect sizes for the injected pathway-level signals

## Outputs

- Semi-synthetic omics DataFrames per effect size with controlled differential abundance in known pathways
- Held-out test AUROC per model (PathIntegrate Multi-View, Single-View, DIABLO molecular-level, DIABLO pathway), effect size, and pathway database
- Fig 4A-style results table (e.g. fig4A_auroc_results.csv) tabulating AUROC across effect sizes

## How to apply

Start from real multi-omics DataFrames (samples as rows, molecule IDs as columns; COPDgene metabolomics and proteomics with ChEBI/UniProt identifiers) and pathway matrices downloaded via the sspa package (KEGG v98 via sspa.process_kegg, Reactome v78 via sspa.process_reactome, using min_entity=2). Preprocess each omics view as in the sspa README: drop molecules with >50% missing values, median-impute, log2-transform, and z-score per molecule. Inject semi-synthetic signal by perturbing molecules belonging to selected pathways with fold changes based on the known outcome (COPD status or gender), sweeping multiple effect sizes so signals are embedded in realistic low signal-to-noise data. Transform each molecular-level dataset to pathway-level scores with ssPA methods (sspa_kpca, sspa_SVD, sspa_ssClustPA, which expose fit/transform for scikit-learn pipelines), then fit PathIntegrate Multi-View (MB-PLS), PathIntegrate Single-View (PLS-DA), and DIABLO baselines at both molecular and pathway levels. Tune the number of latent variables with 5-times repeated nested 5-fold cross-validation and record held-out test AUROC for every model, effect size, and pathway database (KEGG vs Reactome).

## Related tools

- **sspa (py-ssPA)** (Downloads KEGG/Reactome multi-omics pathway databases and provides preprocessing and ssPA transformations used to build and validate the semi-synthetic pathway signals) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Fits Multi-View (MB-PLS) and Single-View (scikit-learn PLS-DA) models on the semi-synthetic pathway-transformed data to obtain benchmark AUROC) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Molecular-level and pathway-level multi-omics baseline model benchmarked against PathIntegrate on the semi-synthetic data)
- **SciKitLearn** (Provides the predictive models, Pipeline and GridSearchCV used for ssPA-transformed data and nested cross-validated tuning of latent variables)
- **PathIntegrate_scripts** (Benchmarking scripts (developed for a PBS HPC cluster) used to run the semi-synthetic COPDgene experiments) — https://github.com/cwieder/PathIntegrate_scripts

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- The expected pattern is reproduced: pathway-based models (PathIntegrate Multi-View, Single-View, DIABLO pathway) beat DIABLO molecular-level AUROC across ALL injected effect sizes
- At larger effect sizes, KEGG-based pathway models perform marginally better than Reactome-based ones (Fig 4C Reactome-vs-KEGG comparison)
- Held-out AUROC is recorded for every combination of model, effect size, and pathway database under 5-times repeated nested 5-fold CV (tuning only inside inner folds)
- Preprocessing invariant: molecules with >50% missing/NA/zero values removed, median imputation, log2 and z-score standardisation applied identically to real and semi-synthetic versions
- Injected signal is verifiable as differential abundance of molecules in known target pathways, with fold changes consistent with the chosen effect size and outcome (COPD status or gender)

## Limitations

- A key drawback of real multi-omics benchmarks is the lack of appropriate benchmarking data; ideally a dataset would have two or more high-quality omics views, n >= 1000 samples, and validated molecular/pathway-level signals — semi-synthetic injection is the workaround but does not fully satisfy the 'validated by' criterion
- It remains a challenge to compare molecular vs. pathway-level models because the number of differentially abundant molecules per pathway is unknown at any one time, and pathway definitions and sizes vary between databases
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, and un-annotated molecules cannot be incorporated, causing information loss
- Injected signals assume fold-change-based perturbation of known pathways, which may not capture realistic multi-pathway or correlated biological signals

## Evidence

- [abstract] benefit of pathway grouping in low SNR: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [supplementary] pathway transformation enhances sensitivity: "Pathway transformation enhances sensitivity to low signal-to-noise signals (COPDgene semi synthetic data)"
- [supplementary] fold changes used for injection: "Fold changes in COPDgene multi-omics data based on either COPD status or gender outcomes"
- [supplementary] CV scheme for latent variable tuning: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [discussion] ideal benchmarking dataset criteria: "Ideally, a benchmarking dataset would contain two or more high-quality omics views, a large sample size (n ≥ 1000), and known biological signals at the molecular and pathway level validated by"
- [readme] sspa preprocessing recipe: [evidence span withheld — anchor status: not_found]
