---
name: fisher-p-value-combination-testing
description: Reference for combining per-molecule statistical p-values into a single
  pathway-level p-value with Fisher's method when benchmarking pathway-based methods
  against molecular-level signals in low signal-to-noise multi-omics data.
when_to_use_negative:
- When molecules are already transformed to pathway-level scores (ssPA output) - the
  Fisher combination applies to molecular-level p-values only
- When a full comparison to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA,
  IMPaLA) is intended - the article states this was beyond the scope of the work
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3520
tools:
- name: sspa (py-ssPA)
  role: Supplies the pathway definitions (KEGG/Reactome) over which molecular p-values
    are combined and provides the ssPA scoring methods used as the pathway-transformed
    comparator
  repo: github.com/cwieder/py-ssPA
- name: PathIntegrate_scripts
  role: Benchmarking scripts that replicate the semi-synthetic simulations including
    the Fisher-combined molecular-level baseline
  repo: cwieder/PathIntegrate_scripts
- name: SciKitLearn
  role: Compatible framework in which ssPA transformation and the downstream predictive/test
    pipeline are run
  repo: https://github.com/cwieder/PathIntegrate
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:c79e1a4c97946f27b87e7644e8aa15d04730eba3372dadcbe601dea4682b7f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - sspa (py-ssPA)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# fisher-p-value-combination-testing

## Summary

Baseline molecular-level strategy in PathIntegrate's semi-synthetic benchmark: per-molecule group tests (e.g. Mann-Whitney-U) yield one p-value per molecule, which are combined per pathway via Fisher's method so pathway-transformed ssPA scoring can be compared against molecular-level signal detection power at effect sizes 0-1.

## When to use

Reference for combining per-molecule statistical p-values into a single pathway-level p-value with Fisher's method when benchmarking pathway-based methods against molecular-level signals in low signal-to-noise multi-omics data.

## When NOT to use

- When molecules are already transformed to pathway-level scores (ssPA output) - the Fisher combination applies to molecular-level p-values only
- When a full comparison to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) is intended - the article states this was beyond the scope of the work

## Inputs

- Molecular-level abundance matrix (pandas DataFrame, samples as rows, molecules as columns, e.g. COPDgene or COVID-19 metabolomics/proteomics data)
- Pathway definitions from KEGG or Reactome loaded via the sspa package (pathway-to-molecule membership lists)
- Per-molecule p-values from a group test (e.g. Mann-Whitney-U between two outcome groups)
- Semi-synthetic effect size values spanning 0-1 (e.g. 0.25-0.55 low-signal range)

## Outputs

- One Fisher-combined pathway-level p-value per pathway per simulation realisation
- Proportion of significant Fisher-combined p-values per effect size (molecular-level detection power curve)
- Comparison plot against ssPA pathway-transformed detection power (fig2_semi_synthetic_signal_detection.png)

## How to apply

For each simulated dataset, compute a per-molecule significance test (the article's semi-synthetic benchmark uses Mann-Whitney-U tests between the two outcome groups) across all molecules, then combine the molecules' p-values within each pathway definition (KEGG or Reactome, downloaded via the sspa package) using Fisher's combined probability test to yield one pathway-level p-value. This molecular-level combined p-value serves as the comparator to pathway scores produced by ssPA transformation methods (sspa_kpca, sspa_SVD, sspa_ssClustPA), against which Mann-Whitney-U tests are also run. Repeat the simulation over effect sizes spanning 0-1 with signal injected into a single target pathway and count the proportion of significant p-values for both strategies, as reported in Fig 2 and its S1 Fig C panel. The rationale: Fisher's method pools weak evidence from individually non-significant molecules, so the comparison quantifies whether grouping molecules into pathways increases detection power when effect sizes are low (0.25-0.55).

## Related tools

- **sspa (py-ssPA)** (Supplies the pathway definitions (KEGG/Reactome) over which molecular p-values are combined and provides the ssPA scoring methods used as the pathway-transformed comparator) — github.com/cwieder/py-ssPA
- **PathIntegrate_scripts** (Benchmarking scripts that replicate the semi-synthetic simulations including the Fisher-combined molecular-level baseline) — cwieder/PathIntegrate_scripts
- **SciKitLearn** (Compatible framework in which ssPA transformation and the downstream predictive/test pipeline are run) — https://github.com/cwieder/PathIntegrate

## Examples

```
from scipy.stats import fisher_method; combined_p = fisher_method([mwu_p[i] for i in pathway_molecules])
```

## Evaluation signals

- At lower effect sizes (0.25-0.55), the proportion of significant p-values in ssPA pathway-transformed data should exceed that of Fisher-combined molecular-level p-values, matching the article's reported finding
- The combined p-value distribution should yield a higher proportion of significant tests than individual molecular p-values when signal is concentrated in one pathway (Fisher pooling effect)
- The molecular-level curve and the pathway-transformed curve should converge at effect size ~1 (high signal) for both COPDgene and COVID-19 simulations
- Number of pathways tested should match the number of valid pathway memberships after molecule-to-pathway mapping via ChEBI/UniProt identifiers

## Limitations

- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics
- Pathway definitions and sizes vary between databases, making molecular vs pathway-level comparison challenging since the number of differentially abundant molecules per pathway is unknown
- Fisher combination assumes independence of the molecular p-values, which may not hold for correlated omics measurements

## Evidence

- [abstract] Semi-synthetic MWU benchmark: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [results] Fisher combination context: [evidence span withheld — anchor status: not_found]
- [readme] ssPA transformation methods: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark scripts availability: [evidence span withheld — anchor status: external_doc]
- [discussion] Pathway coverage bottleneck: [evidence span withheld — anchor status: not_found]
