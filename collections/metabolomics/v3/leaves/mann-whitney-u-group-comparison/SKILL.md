---
name: mann-whitney-u-group-comparison
description: Use when comparing two groups (e.g. COPD vs control, mild vs severe COVID-19)
  on pathway-score or molecular-level matrices to estimate what fraction of features
  separates groups under controlled effect sizes.
when_to_use_negative:
- When the goal is predictive multi-omics modelling rather than univariate power assessment
  — PathIntegrate Single-View or Multi-View supervised models are the appropriate
  framework.
- When molecules cannot be reliably mapped to pathways (poor pathway coverage/annotation),
  since the advantage of grouping molecules into pathways depends on usable pathway
  definitions.
- When comparing more than two groups or needing multivariate interaction modelling
  — MWU is a two-sample univariate test.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: sspa (py-ssPA)
  role: Fetch KEGG/Reactome pathway definitions and transform molecular-level matrices
    into ssPA pathway scores for MWU testing
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Framework within which the semi-synthetic univariate MWU benchmark of pathway
    vs molecular signal detection is executed
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Compatible ML functions used alongside ssPA transformations for model optimisation
    and evaluation in the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:3ef71081e5738b0af84708aa551ac33448bd593e7b733e36f15ef8628c34dbb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa (py-ssPA)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mann-whitney-u-group-comparison

## Summary

Applies Mann-Whitney-U (MWU) univariate tests to ssPA pathway-transformed multi-omics scores to quantify group-separation power, as used in PathIntegrate's semi-synthetic benchmarks where pathway transformation detected more significant signals than molecular-level Fisher-combined testing at effect sizes 0.25–0.55.

## When to use

Use when comparing two groups (e.g. COPD vs control, mild vs severe COVID-19) on pathway-score or molecular-level matrices to estimate what fraction of features separates groups under controlled effect sizes.

## When NOT to use

- When the goal is predictive multi-omics modelling rather than univariate power assessment — PathIntegrate Single-View or Multi-View supervised models are the appropriate framework.
- When molecules cannot be reliably mapped to pathways (poor pathway coverage/annotation), since the advantage of grouping molecules into pathways depends on usable pathway definitions.
- When comparing more than two groups or needing multivariate interaction modelling — MWU is a two-sample univariate test.

## Inputs

- COPDgene and COVID-19 metabolomics and proteomics abundance matrices (pandas DataFrames, samples as rows, molecules as columns)
- Pathway definitions from KEGG/Reactome fetched via the sspa package
- Simulated effect-size grid spanning 0–1 for signal injection into a target pathway

## Outputs

- Proportion of significant Mann-Whitney-U p-values per effect size for pathway-transformed (ssPA) and molecular-level (Fisher-combined) approaches
- Plot of proportion significant p-values against effect size (fig2_semi_synthetic_signal_detection.png)

## How to apply

Load the metabolomics/proteomics abundance matrices (samples as rows, molecules as columns) and pathway definitions from KEGG/Reactome via the sspa package, then simulate semi-synthetic datasets by enriching one target pathway per realisation with injected signal at effect sizes spanning 0–1, leaving remaining molecules as background noise. Transform each simulated dataset to pathway level with single-sample pathway analysis (ssPA) scores, and in parallel compute molecular-level differential abundance p-values combined across molecules with Fisher's method. Run one Mann-Whitney-U test per pathway per realisation on the ssPA scores, and one on the Fisher-combined molecular p-values. For each effect size, compute the proportion of significant MWU p-values under the pathway-transformed versus molecular-level strategies and plot proportion significant against effect size (reproducing Fig 2 / S1 Fig C, saved as fig2_semi_synthetic_signal_detection.png). Validation succeeds if the ssPA pathway scores yield a higher proportion of significant MWU p-values than Fisher-combined molecular testing in the low effect-size range 0.25–0.55, matching the reported finding that grouping molecules into pathways benefits signal detection in low signal-to-noise scenarios.

## Related tools

- **sspa (py-ssPA)** (Fetch KEGG/Reactome pathway definitions and transform molecular-level matrices into ssPA pathway scores for MWU testing) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Framework within which the semi-synthetic univariate MWU benchmark of pathway vs molecular signal detection is executed) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Compatible ML functions used alongside ssPA transformations for model optimisation and evaluation in the benchmark)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- In the low effect-size range (0.25–0.55), the proportion of significant MWU p-values on ssPA pathway-transformed data exceeds that on Fisher-combined molecular-level data.
- At effect size 0 (no signal), the proportion of significant p-values approaches the nominal false-positive rate.
- Both COPDgene and COVID-19 simulations show the same monotonic relationship between effect size and proportion of significant MWU p-values, as in Fig 2 / S1 Fig C.
- Each realisation has exactly one target pathway enriched, with remaining molecules kept as background noise.
- One MWU test is performed per pathway per realisation, and the number of tests is identical between pathway-transformed and molecular-level strategies.

## Limitations

- It remains a challenge to compare molecular vs. pathway-level models because it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes vary between databases.
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics.
- The semi-synthetic design cannot fully replicate real biological heterogeneity; results are conditioned on the chosen effect-size grid and injected pathway.

## Evidence

- [results] At lower effect sizes (0.25–0.55), univariate MWU testing detected a higher proportion of significant p-values in ssPA pathway-transformed data than in molecular-level data (with Fisher-combined molecular p-values).: [evidence span withheld — anchor status: not_found]
- [abstract] Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [abstract] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [discussion] We applied PathIntegrate to two datasets: COPDgene and COVID19 multi-omics.: "We applied PathIntegrate to two datasets: COPDgene and COVID19 multi-omics."
- [readme] sspa provides a Python interface for metabolomics pathway analysis... it also provides a wide range of single-sample pathway analysis (ssPA) methods.: [evidence span withheld — anchor status: external_doc]
- [readme] All ssPA methods now have a fit(), transform() and fit_transform() method for compatibility with SciKitLearn.: [evidence span withheld — anchor status: external_doc]
