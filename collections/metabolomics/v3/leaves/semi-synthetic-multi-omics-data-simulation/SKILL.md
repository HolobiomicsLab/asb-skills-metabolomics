---
name: semi-synthetic-multi-omics-data-simulation
description: Use when benchmarking multi-omics integration or pathway methods on real
  data that lacks validated molecular/pathway-level ground truth, by injecting a known
  enriched-pathway signal into real abundance matrices at controlled effect sizes.
when_to_use_negative:
- When a genuine benchmarking dataset with validated molecular and pathway-level signals
  already exists — the paper notes ideal benchmarks need two or more high-quality
  omics views, n ≥ 1000, and known signals, in which case simulation is unnecessary.
- When the goal is prediction or interpretation on the real data itself rather than
  method benchmarking — injecting synthetic signals would corrupt the biological outcomes
  being modelled.
- When evaluating methods that operate purely at the molecular level without pathway
  context if no pathway-based comparator is needed — the simulation's target-pathway
  design is built around pathway-level detection questions.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Python package whose Multi-View (MB-PLS/MB-VIP) and Single-View models and
    benchmarking scripts are evaluated on the semi-synthetic data; DOI 10.5281/zenodo.14060723
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Downloads KEGG/Reactome pathway definitions and applies single-sample pathway
    analysis (ssPA) transformations to the simulated molecular matrices before detection
    testing
  repo: https://github.com/cwieder/py-ssPA
- name: DIABLO
  role: Benchmarked pathway-based integration method whose pathway relevance scores
    (RGCCA loadings) and sparse-L1 non-zero-loading statistics are compared with PathIntegrate
    on the simulated data
- name: SciKitLearn
  role: Compatibility layer for model optimisation/evaluation (e.g. Pipeline, GridSearchCV)
    during benchmarking on simulated datasets
- name: PathIntegrate_scripts
  role: Benchmarking and application scripts repository; COVID-19 analyses by Su et
    al. 2020 can be replicated from code and data within it
  repo: https://github.com/cwieder/PathIntegrate_scripts
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:46a0cdaa7c749b82cc295159daf495d44066368fbe793becb0f8927ee55419a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - sspa
  - sspa (py-ssPA)
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# semi-synthetic-multi-omics-data-simulation

## Summary

A benchmarking skill from the PathIntegrate paper: generate semi-synthetic multi-omics datasets by injecting a pathway-enriched signal at controlled effect sizes (0–1) into real metabolomics/proteomics abundance matrices, then score how well molecular- vs pathway-level methods detect the signal.

## When to use

Use when benchmarking multi-omics integration or pathway methods on real data that lacks validated molecular/pathway-level ground truth, by injecting a known enriched-pathway signal into real abundance matrices at controlled effect sizes.

## When NOT to use

- When a genuine benchmarking dataset with validated molecular and pathway-level signals already exists — the paper notes ideal benchmarks need two or more high-quality omics views, n ≥ 1000, and known signals, in which case simulation is unnecessary.
- When the goal is prediction or interpretation on the real data itself rather than method benchmarking — injecting synthetic signals would corrupt the biological outcomes being modelled.
- When evaluating methods that operate purely at the molecular level without pathway context if no pathway-based comparator is needed — the simulation's target-pathway design is built around pathway-level detection questions.

## Inputs

- Real multi-omics abundance matrices (metabolomics and proteomics, e.g. COPDgene; COVID-19 Su et al. 2020) as pandas DataFrames (samples × molecules)
- Pathway definitions (KEGG or Reactome human databases) as GMT-like dictionaries from sspa (sspa.process_reactome, sspa.process_kegg)
- Simulation parameters: target pathway ID, per-realisation effect sizes spanning 0–1, number of realisations
- Group/label column for the semi-synthetic outcome (e.g. COPD status or COVID-19 mild vs severe)

## Outputs

- Semi-synthetic multi-omics datasets with embedded target-pathway signals at each effect size
- Per-strategy detection statistics: proportions of significant Mann-Whitney U p-values (pathway-transformed vs Fisher-combined molecular) and confusion-matrix AUCs of pathway importance scores
- Figures such as fig2_semi_synthetic_signal_detection.png (proportion significant vs effect size, reproducing Fig 2 / S1 Fig C)
- AUC results table pathway_detection_auc.csv (method × effect size)

## How to apply

Load real multi-omics abundance matrices (e.g. COPDgene or COVID-19 metabolomics/proteomics from Su et al. 2020) as pandas DataFrames with samples as rows and molecules as columns, and fetch pathway definitions from KEGG or Reactome via the sspa package. For each realisation, enrich a single target pathway by injecting signal at effect sizes spanning 0–1 while leaving remaining molecules as background noise, creating the low signal-to-noise scenarios the paper targets. Evaluate detection by testing group separation with Mann-Whitney U tests on ssPA pathway scores versus Fisher-combined molecular p-values, and/or by fitting PathIntegrate Multi-View (MB-VIP importance with 10,000 permutations and BH FDR correction), DIABLO, and sparse DIABLO models, then compute confusion-matrix AUCs for target-pathway detection. The rationale is that real data lack a validated benchmarking dataset with high-quality omics views, large sample size (n ≥ 1000), and known molecular and pathway-level signals, so semi-synthetic injection supplies the ground truth needed to compare pathway-transformation against molecular-level modelling.

## Related tools

- **PathIntegrate** (Python package whose Multi-View (MB-PLS/MB-VIP) and Single-View models and benchmarking scripts are evaluated on the semi-synthetic data; DOI 10.5281/zenodo.14060723) — https://github.com/cwieder/PathIntegrate
- **sspa** (Downloads KEGG/Reactome pathway definitions and applies single-sample pathway analysis (ssPA) transformations to the simulated molecular matrices before detection testing) — https://github.com/cwieder/py-ssPA
- **DIABLO** (Benchmarked pathway-based integration method whose pathway relevance scores (RGCCA loadings) and sparse-L1 non-zero-loading statistics are compared with PathIntegrate on the simulated data)
- **SciKitLearn** (Compatibility layer for model optimisation/evaluation (e.g. Pipeline, GridSearchCV) during benchmarking on simulated datasets)
- **PathIntegrate_scripts** (Benchmarking and application scripts repository; COVID-19 analyses by Su et al. 2020 can be replicated from code and data within it) — https://github.com/cwieder/PathIntegrate_scripts

## Examples

```
reactome_pathways = sspa.process_reactome(organism="Homo sapiens"); kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(sim_data)
```

## Evaluation signals

- At low effect sizes (0.25–0.55), the ssPA pathway-transformed data should yield a higher proportion of significant Mann-Whitney U p-values than Fisher-combined molecular-level p-values (expected from paper's finding).
- PathIntegrate Multi-View/Single-View and DIABLO pathway models should reach confusion-matrix detection AUC ≥ 0.90 for the enriched target pathway even at effect size 0.25 in high-noise scenarios.
- Null detection statistics must come from permutation-based distributions (10,000 permutations) with Benjamini-Hochberg FDR correction — check the confusion matrices were built this way.
- The effect-size sweep must span 0–1 with signal injected into exactly one target pathway per realisation; background molecules remain unmodified.
- The effect-size–versus-detection curve for pathway-transformed methods should rise above the molecular-level curve, matching Fig 2 and S1 Fig C behaviour on both COPDgene and COVID-19 simulations.

## Limitations

- Lack of appropriate benchmarking data: no real dataset meeting the ideal criteria (two or more high-quality omics views, n ≥ 1000, and validated molecular/pathway-level signals) was available, which is why semi-synthetic data are used.
- It remains a challenge to compare molecular vs. pathway-level models because it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes vary between databases.
- Pathway coverage (the proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, affecting how much signal can be injected per pathway.
- The COVID-19 simulations can be fully replicated from cwieder/PathIntegrate_scripts, but COPDgene simulations require access to the COPDGene consortium data (dbGaP phs000179.v6.p2, phs000765.v3.p2, Metabolomics Workbench PR000907).

## Evidence

- [abstract] Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [discussion] a key drawback being the lack of appropriate benchmarking data: "a key drawback being the lack of appropriate benchmarking data"
- [discussion] Ideally, a benchmarking dataset would contain two or more high-quality omics views, a large sample size (n ≥ 1000), and known biological signals at the molecular and pathway level validated by: "Ideally, a benchmarking dataset would contain two or more high-quality omics views, a large sample size (n ≥ 1000), and known biological signals at the molecular and pathway level validated by"
- [discussion] it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant at any one time: "it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes"
- [readme] COVID_data: contains the COVID-19 metabolomics and proteomics data from Su et al. 2020: [evidence span withheld — anchor status: external_doc]
