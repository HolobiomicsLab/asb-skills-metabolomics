---
name: pathway-level-vs-molecular-level-signal-detection-benchmarking
description: Use when you need to empirically justify a pathway-transformation step
  by comparing pathway-level versus molecular-level univariate detection power on
  injected-signal simulations spanning effect sizes 0–1.
when_to_use_negative:
- Your goal is a full comparison of PathIntegrate to conventional pathway analysis
  approaches (ORA, GSEA, MultiGSEA, IMPaLA) — the authors explicitly state this was
  beyond the scope of their work
- You need a benchmarking dataset with known validated molecular/pathway-level signals
  and n ≥ 1000 — the authors note such an ideal benchmarking dataset was not available,
  so semi-synthetic injection is a proxy rather than a validation of real biological
  signals
- Your comparison depends on exact pathway definitions and sizes — the authors warn
  these vary between databases, making molecular vs. pathway-level model comparison
  a challenge
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3520
tools:
- name: sspa
  role: Fetches KEGG/Reactome pathway definitions and provides single-sample pathway
    analysis (ssPA) scoring methods to transform molecular-level matrices into pathway-level
    matrices
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: ssPA methods expose fit/transform/fit_transform for compatibility with SciKitLearn,
    enabling Pipeline/GridSearchCV-style integration of the transformation step
- name: Python
  role: Language in which the simulation, transformation, MWU testing, and plotting
    workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:1fdc19ca529b666163cb2ec8902552ad15634dbc5215ea326e20642ad284d3e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  - sspa (py-ssPA)
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# pathway-level-vs-molecular-level-signal-detection-benchmarking

## Summary

A semi-synthetic benchmarking skill that quantifies whether single-sample pathway analysis (ssPA) transformation of multi-omics abundance matrices increases univariate Mann-Whitney-U detection power versus Fisher-combined molecular-level tests in low signal-to-noise scenarios.

## When to use

Use when you need to empirically justify a pathway-transformation step by comparing pathway-level versus molecular-level univariate detection power on injected-signal simulations spanning effect sizes 0–1.

## When NOT to use

- Your goal is a full comparison of PathIntegrate to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) — the authors explicitly state this was beyond the scope of their work
- You need a benchmarking dataset with known validated molecular/pathway-level signals and n ≥ 1000 — the authors note such an ideal benchmarking dataset was not available, so semi-synthetic injection is a proxy rather than a validation of real biological signals
- Your comparison depends on exact pathway definitions and sizes — the authors warn these vary between databases, making molecular vs. pathway-level model comparison a challenge

## Inputs

- Multi-omics abundance matrices as pandas DataFrames (samples as rows, molecules as columns), e.g. COPDgene and COVID-19 (Su et al. 2020) metabolomics and proteomics data
- Pathway definitions from KEGG or Reactome fetched via the sspa package
- Injected signal effect sizes spanning 0–1 with one target pathway enriched per realisation

## Outputs

- Proportion of significant Mann-Whitney-U p-values per effect size for pathway-transformed (ssPA) and Fisher-combined molecular-level approaches
- Comparison plot of proportion significant vs. effect size reproducing Fig 2 / S1 Fig C (fig2_semi_synthetic_signal_detection.png)
- Mann-Whitney-U p-values for each pathway per realisation and for Fisher-combined molecular p-values

## How to apply

Load real multi-omics abundance matrices (e.g. the COPDgene and COVID-19 metabolomics/proteomics data from Su et al. 2020) as pandas DataFrames with samples as rows and molecules as columns, and fetch pathway definitions from KEGG or Reactome via the sspa package. Simulate semi-synthetic realisations by injecting signal into a single target pathway per realisation at effect sizes spanning 0–1 while the remaining molecules act as background noise, creating low signal-to-noise scenarios. Transform each simulated dataset to pathway level using ssPA scoring methods from sspa, and in parallel compute molecular-level differential abundance p-values combined across molecules with Fisher's method. Run Mann-Whitney-U tests on the ssPA pathway scores (one test per pathway per realisation) and on the Fisher-combined molecular p-values, then plot the proportion of significant p-values against effect size for both strategies. The benchmark validates when ssPA-transformed data show a higher proportion of significant MWU p-values than the molecular-level approach in the low effect-size range (0.25–0.55), reproducing Fig 2 and its S1 Fig C panel as fig2_semi_synthetic_signal_detection.png.

## Related tools

- **sspa** (Fetches KEGG/Reactome pathway definitions and provides single-sample pathway analysis (ssPA) scoring methods to transform molecular-level matrices into pathway-level matrices) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (ssPA methods expose fit/transform/fit_transform for compatibility with SciKitLearn, enabling Pipeline/GridSearchCV-style integration of the transformation step)
- **Python** (Language in which the simulation, transformation, MWU testing, and plotting workflow is executed)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2]); mwu_pvals = scipy.stats.mannwhitneyu(kpca_scores.values, covid_data_processed['Group'])  # Fisher-combined molecular p-values computed in parallel with scipy.stats.fisher_exact
```

## Evaluation signals

- In the low effect-size range (0.25–0.55), the proportion of significant MWU p-values on ssPA-transformed data exceeds that on Fisher-combined molecular-level data, matching the reported finding that grouping molecules into pathways benefits signal detection in low signal-to-noise scenarios
- The proportion-significant vs. effect-size curves are plotted for both strategies on COPDgene and COVID-19 simulations, reproducing Fig 2 and S1 Fig C panel shape
- Each simulated realisation has exactly one target pathway enriched with injected signal while remaining molecules serve as background noise
- Molecular-level p-values are combined across molecules with Fisher's method before MWU testing, ensuring the molecular baseline is computed on the same realisations as the ssPA scores
- The output figure is saved as fig2_semi_synthetic_signal_detection.png

## Limitations

- The benchmark lacks an ideal dataset with two or more high-quality omics views, large sample size (n ≥ 1000), and known validated molecular/pathway-level signals — a key drawback being the lack of appropriate benchmarking data
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, affecting how much of the injected pathway signal is measurable
- It remains a challenge to compare molecular vs. pathway-level models because it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes vary between databases

## Evidence

- [results] ssPA transformation yields higher MWU detection power than molecular-level Fisher combination at effect sizes 0.25–0.55: [evidence span withheld — anchor status: not_found]
- [abstract] Grouping molecules into pathways benefits signal detection in low signal-to-noise scenarios: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [intro] ssPA transforms molecular abundance matrices into pathway-level matrices: "PathIntegrate employs single-sample pathway analysis approaches (ssPA) (Fig 1), which transform molecular-level abundance data matrices into pathway-level matrices"
- [readme] sspa provides pathway database download (KEGG, Reactome, PathBank) and identifier conversion: [evidence span withheld — anchor status: external_doc]
- [readme] ssPA methods are SciKitLearn-compatible for pipeline integration: [evidence span withheld — anchor status: external_doc]
- [readme] PathIntegrate scripts can replicate COVID-19 analyses; COPDgene requires consortium access: [evidence span withheld — anchor status: external_doc]
