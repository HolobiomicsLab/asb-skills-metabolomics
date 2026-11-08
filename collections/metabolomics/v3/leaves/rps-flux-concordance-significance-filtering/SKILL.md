---
name: rps-flux-concordance-significance-filtering
description: Use when a concordance table of Cohen's kappa and Pearson correlation
  scores (RPS vs RAS and RPS vs FFD) exists and you need to determine which reactions'
  RPS-FFD concordance survives multiple-testing correction.
when_to_use_negative:
- The concordance table has not yet been computed — this skill presupposes output
  of the concordanceAnalysis step, not raw metabolomics or sampling data
- The goal is to classify flux regulation by intersecting transcriptomics-based (RAS)
  and metabolomics-based (FFD) predictions, rather than to statistically test kappa
  concordance significance
- The analysis concerns RPS vs RAS (transcriptional-layer) concordance without a multiple-comparison
  correction requirement
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: INTEGRATE
  role: Pipeline providing the concordanceAnalysis step and downstream significance
    testing context for RPS vs FFD kappa scores
  repo: github.com/qLSLab/integrate
- name: Python
  role: Runtime for executing the pipeline scripts that compute concordance scores,
    bootstrap null distributions, and BH-adjusted p-values
  repo: github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:0483af8c38efa1fc168a0f7198d47389e953b7e8c631d4591ac848f1a182b174
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# RPS flux concordance significance filtering

## Summary

Assigns empirical p-values to Cohen's kappa concordance scores between Reaction Propensity Scores (RPSs) and FFD-derived flux variations via bootstrap resampling, then applies Benjamini-Hochberg correction at 5% FDR to identify reactions with statistically significant RPS-vs-FFD concordance.

## When to use

Use when a concordance table of Cohen's kappa and Pearson correlation scores (RPS vs RAS and RPS vs FFD) exists and you need to determine which reactions' RPS-FFD concordance survives multiple-testing correction.

## When NOT to use

- The concordance table has not yet been computed — this skill presupposes output of the concordanceAnalysis step, not raw metabolomics or sampling data
- The goal is to classify flux regulation by intersecting transcriptomics-based (RAS) and metabolomics-based (FFD) predictions, rather than to statistically test kappa concordance significance
- The analysis concerns RPS vs RAS (transcriptional-layer) concordance without a multiple-comparison correction requirement

## Inputs

- Concordance table (Cohen's kappa with 'linear' weighting and Pearson correlation) of RPS vs RAS and RPS vs FFD per reaction, from concordanceAnalysis
- RPS pairwise-variation distributions per reaction (used as the bootstrap resampling population)
- Metabolomic statistical test outputs from createMetabolicDataset (e.g. resultsMetabolomic prefix files and medie_Met.csv) feeding the concordance analysis

## Outputs

- adjusted_pvalues.csv containing per-reaction empirical p-values and Benjamini-Hochberg adjusted p-values
- Filtered list of reactions with adjusted p-value < 0.05 (expected: ACONT, ASPTA (forward), RPI, RPE)

## How to apply

Load the concordance table produced by the upstream concordanceAnalysis step, which scores each reaction with the Cohen's kappa coefficient ('linear' weighting) and Pearson correlation of RPS vs RAS and RPS vs FFD. For each reaction, bootstrap-resample the RPS pairwise-variation distribution 1000 times with replacement to build an empirical null distribution of the kappa concordance score, then assign an empirical p-value as the fraction of null kappa values at least as extreme as the observed score. Apply the Benjamini-Hochberg procedure to these empirical p-values at a 5% false discovery rate to obtain adjusted p-values. Filter to reactions whose adjusted p-value falls below the 5% FDR threshold and save the p-value/adjusted-p-value table as adjusted_pvalues.csv. The rationale is that raw kappa concordance scores alone cannot distinguish real substrate-availability-driven flux control from chance agreement across the many reactions tested.

## Related tools

- **INTEGRATE** (Pipeline providing the concordanceAnalysis step and downstream significance testing context for RPS vs FFD kappa scores) — github.com/qLSLab/integrate
- **Python** (Runtime for executing the pipeline scripts that compute concordance scores, bootstrap null distributions, and BH-adjusted p-values) — github.com/qLSLab/integrate

## Examples

```
python pipeline/concordanceAnalysis.py
```

## Evaluation signals

- After 5% FDR correction, the significant reaction set must contain ACONT, ASPTA (forward), RPI, and RPE
- Kappa concordance scores must be computed with 'linear' weighting, matching the concordanceAnalysis default
- Empirical p-values must derive from 1000 bootstrap resamples with replacement of the RPS pairwise-variation distribution
- adjusted_pvalues.csv must contain both raw empirical p-values and BH-adjusted p-values, with adjusted values >= raw values
- All retained reactions must have adjusted p-value strictly below the 0.05 FDR threshold

## Limitations

- Empirical p-value resolution is bounded by the number of bootstrap resamples (1000), so the minimum achievable p-value is on the order of 0.001
- Very small ratios near zero are handled with eps = 0.00001 in the upstream concordance computation to avoid undefined ratios, which can affect borderline concordance scores
- Validation is specific to the ENGRO2 breast cell-line analysis; other models or datasets will yield a different significant-reaction set and need independent validation

## Evidence

- [readme] A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues: "A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues"
- [readme] python pipeline/concordanceAnalysis.py: "Usage: `python pipeline/concordanceAnalysis.py`"
- [intro] INTEGRATE uses intracellular metabolomics data and the mass action law formulation to predict how differences in substrate availability translate into differences in metabolic fluxes (metabolic: [evidence span withheld — anchor status: not_found]
