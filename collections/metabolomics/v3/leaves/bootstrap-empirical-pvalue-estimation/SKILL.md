---
name: bootstrap-empirical-pvalue-estimation
description: Use when a concordance table of Cohen's kappa and Pearson correlation
  scores (e.g. RPS vs RAS, RPS vs FFD) is available and per-reaction significance
  of concordance must be assessed without a parametric null model.
when_to_use_negative:
- When a parametric significance test is appropriate for the concordance statistic
  and a proper analytic null exists — the bootstrap here builds an empirical null
  specifically because kappa has none
- When the input is the raw metabolomics/transcriptomics dataset rather than the downstream
  concordance table from concordanceAnalysis (upstream steps like createMetabolicDataset
  or rasTtest should run first)
tools:
- name: INTEGRATE
  role: Pipeline whose concordanceAnalysis step produces the kappa/Pearson concordance
    table that this bootstrap-FDR correction step consumes
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Language used to implement the bootstrap resampling (1000 replicates with
    replacement) and the Benjamini-Hochberg adjustment
  repo: https://github.com/qLSLab/integrate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:ad31bd1b17a15c26c4e5e4fbd4891a02e5f8946d8179162e3849cb725821bcbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - INTEGRATE
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bootstrap-empirical-pvalue-estimation

## Summary

Assigns empirical p-values to Cohen's kappa concordance scores between RPS and RAS/FFD reaction-variation distributions by bootstrap-resampling the RPS pairwise-variation distribution 1000 times, then applies the Benjamini-Hochberg procedure at 5% FDR to identify reactions with statistically significant concordance.

## When to use

Use when a concordance table of Cohen's kappa and Pearson correlation scores (e.g. RPS vs RAS, RPS vs FFD) is available and per-reaction significance of concordance must be assessed without a parametric null model.

## When NOT to use

- When a parametric significance test is appropriate for the concordance statistic and a proper analytic null exists — the bootstrap here builds an empirical null specifically because kappa has none
- When the input is the raw metabolomics/transcriptomics dataset rather than the downstream concordance table from concordanceAnalysis (upstream steps like createMetabolicDataset or rasTtest should run first)

## Inputs

- Concordance table of Cohen's kappa ('linear' weighting) and Pearson correlation scores of RPS vs RAS and RPS vs FFD, produced by concordanceAnalysis
- RPS pairwise-variation distributions for each reaction (input to the bootstrap resampling)

## Outputs

- Empirical p-value per reaction (fraction of bootstrap null kappa values at least as extreme as the observed score)
- Benjamini-Hochberg adjusted p-values at 5% FDR, saved as adjusted_pvalues.csv
- Filtered list of reactions with adjusted p-value below the 5% FDR threshold

## How to apply

Load the concordance table produced by the upstream concordanceAnalysis step, which computed Cohen's kappa with 'linear' weighting and Pearson correlation of RPS vs RAS and RPS vs FFD. For each reaction, bootstrap-resample the RPS pairwise-variation distribution 1000 times with replacement to build an empirical null distribution of the kappa concordance score. Assign an empirical p-value as the fraction of bootstrap null kappa values at least as extreme as the observed score. Apply the Benjamini-Hochberg procedure to the empirical p-values at a 5% false discovery rate, then filter to reactions whose adjusted p-value is below the 5% FDR threshold. Rationale: kappa concordance scores have no closed-form null distribution, so resampling the observed pairwise-variation distribution provides a reaction-specific empirical null.

## Related tools

- **INTEGRATE** (Pipeline whose concordanceAnalysis step produces the kappa/Pearson concordance table that this bootstrap-FDR correction step consumes) — https://github.com/qLSLab/integrate
- **Python** (Language used to implement the bootstrap resampling (1000 replicates with replacement) and the Benjamini-Hochberg adjustment) — https://github.com/qLSLab/integrate

## Examples

```
python pipeline/concordanceAnalysis.py  # upstream step producing the kappa/Pearson concordance table consumed by the bootstrap p-value correction
```

## Evaluation signals

- Significant reactions after FDR correction must include ACONT, ASPTA (forward), RPI, and RPE
- adjusted_pvalues.csv must contain both raw empirical p-values and BH-adjusted p-values for every reaction in the concordance table
- Bootstrap must use 1000 resamples with replacement of the RPS pairwise-variation distribution, matching the kappa weighting ('linear') used in the concordance analysis
- Adjusted p-values must be monotone non-decreasing relative to raw p-values and no smaller than the raw p-values (BH property)
- Filtered output must contain only reactions with adjusted p-value < 0.05

## Limitations

- Empirical p-values are resolution-limited by the 1000-replicate bootstrap, bounding the smallest attainable p-value
- The null distribution is reaction-specific and derived from resampling the observed RPS pairwise variations, so it assumes the observed variation distribution is exchangeable under the null
- Significance is only defined for the concordance statistic actually computed upstream; changing the kappa weighting (default 'linear') changes the null and the result

## Evidence

- [results] After FDR correction of the p-values, the concordance between RPSs and FFDs resulted statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE.: "After FDR correction of the p-values, the concordance between RPSs and FFDs resulted statistically significant for reactions ACONT, ASPTA (forward), RPI, and RPE."
- [methods] bootstrap-resample the RPS pairwise-variation distribution 1000 times with replacement to build an empirical null distribution of the Cohen's kappa concordance score: [evidence span withheld — anchor status: not_found]
- [readme] A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues: "A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues"
- [readme] weight: weight on the Kappa Cohen. Default value: 'linear': "weight: weight on the Kappa Cohen. Default value: 'linear'"
