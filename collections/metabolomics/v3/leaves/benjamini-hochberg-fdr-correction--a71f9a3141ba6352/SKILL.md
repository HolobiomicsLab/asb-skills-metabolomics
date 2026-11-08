---
name: benjamini-hochberg-fdr-correction
description: Use when a table of per-reaction concordance scores (Cohen's kappa, Pearson
  correlation of RPS vs RAS and RPS vs FFD) needs multiple-comparison correction to
  decide which reactions are statistically significantly concordant at 5% FDR.
when_to_use_negative:
- When raw p-values already exist from a parametric test that accounts for multiplicity
  (e.g. the Mann-Whitney U or t-test outputs of steps 7-8) rather than empirical concordance
  scores
- When the upstream concordance table was not computed with Cohen's kappa ('linear'
  weighting) — the bootstrap null construction assumes the same kappa score as the
  concordance analysis
- When the analysis goal is identifying transcriptionally regulated reactions (RPS
  vs RAS only) rather than metabolite-level concordance significance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: INTEGRATE
  role: Pipeline whose concordanceAnalysis step produces the kappa/Pearson concordance
    table that the bootstrap empirical p-value and BH correction operate on
  repo: github.com/QLSLab/integrate
- name: Python
  role: Language used to run the pipeline scripts, including concordanceAnalysis.py
    and the bootstrap/BH correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:02177545ff6819a903ef087ae1089b4231909dc196d1822f8ba295788fe3013d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# benjamini-hochberg-fdr-correction

## Summary

Statistical correction of reaction concordance significance testing: assign empirical p-values to Cohen's kappa concordance scores (RPS vs FFD/RAS) via 1000 bootstrap resamples, then apply Benjamini-Hochberg adjustment at 5% FDR to identify significantly concordant metabolic reactions.

## When to use

Use when a table of per-reaction concordance scores (Cohen's kappa, Pearson correlation of RPS vs RAS and RPS vs FFD) needs multiple-comparison correction to decide which reactions are statistically significantly concordant at 5% FDR.

## When NOT to use

- When raw p-values already exist from a parametric test that accounts for multiplicity (e.g. the Mann-Whitney U or t-test outputs of steps 7-8) rather than empirical concordance scores
- When the upstream concordance table was not computed with Cohen's kappa ('linear' weighting) — the bootstrap null construction assumes the same kappa score as the concordance analysis
- When the analysis goal is identifying transcriptionally regulated reactions (RPS vs RAS only) rather than metabolite-level concordance significance

## Inputs

- Concordance table of Cohen's kappa and Pearson correlation scores of RPS vs RAS and RPS vs FFD (from concordanceAnalysis step)
- RPS pairwise-variation distributions per reaction for bootstrap null construction
- Kappa weighting parameter set to 'linear'

## Outputs

- adjusted_pvalues.csv with empirical p-values and Benjamini-Hochberg adjusted p-values per reaction
- Filtered list of reactions with adjusted p-value below the 5% FDR threshold

## How to apply

Load the concordance table produced by the upstream concordanceAnalysis step (Cohen's kappa and Pearson correlation of RPS vs RAS and RPS vs FFD). For each reaction, bootstrap-resample the RPS pairwise-variation distribution 1000 times with replacement to build an empirical null distribution of the kappa concordance score, using the 'linear' kappa weighting as in the concordance analysis. Compute each empirical p-value as the fraction of bootstrap null kappa values at least as extreme as the observed score. Apply the Benjamini-Hochberg procedure to the empirical p-values at a 5% false discovery rate, keep reactions whose adjusted p-value falls below the threshold, and write the p-value/adjusted-p-value table as adjusted_pvalues.csv. In the INTEGRATE pipeline this correction runs downstream of concordanceAnalysis.py, which itself consumes the metabolomic test files (resultsMetabolomic prefix, means file medie_Met.csv, valLog threshold 1.2, eps 0.00001) to avoid undefined ratios.

## Related tools

- **INTEGRATE** (Pipeline whose concordanceAnalysis step produces the kappa/Pearson concordance table that the bootstrap empirical p-value and BH correction operate on) — github.com/QLSLab/integrate
- **Python** (Language used to run the pipeline scripts, including concordanceAnalysis.py and the bootstrap/BH correction)

## Examples

```
python pipeline/concordanceAnalysis.py
```

## Evaluation signals

- After correction at 5% FDR, the significant reactions must be exactly ACONT, ASPTA (forward), RPI, and RPE
- adjusted_pvalues.csv must contain both the raw empirical p-values and the BH adjusted p-values for every reaction in the concordance table
- Each empirical p-value must equal the fraction of the 1000 bootstrap null kappa values at least as extreme as the observed score
- The kappa null distributions must be built with the same 'linear' weighting used in concordanceAnalysis
- No reaction with adjusted p-value >= 0.05 may appear in the filtered significant set

## Limitations

- Empirical p-values depend on 1000 bootstrap resamples — with few replicas (pipeline default lReplicas ['_A','_B']) the null may be coarse
- The correction is specific to the Cohen's kappa concordance formulation with 'linear' weighting; other concordance metrics would need a different null construction
- Only the concordance of RPS vs FFD (and RPS vs RAS) scores is tested; the intersection classification of metabolic vs transcriptional regulation is done by a separate upstream step

## Evidence

- [readme] A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues: "A dataset of concordance analysis (Cohen coefficient and pearson correlation) of RPS vs RAS, RPS vs FFD, RPS vs FFD, the pvalues and the adjusted pvalues"
- [readme] weight: weight on the Kappa Cohen. Default value: 'linear': "weight: weight on the Kappa Cohen. Default value: 'linear'"
- [readme] eps: a very close to 0 value to avoid undefined ratios. Default value: 0.00001.: "eps: a very close to 0 value to avoid undefined ratios. Default value: 0.00001."
- [readme] Step 9: Create metabolomic statistical test dataset Aim: create the metabolomic dataset to perform concordance analysis: "Step 9: Create metabolomic statistical test dataset Aim: create the metabolomic dataset to perform concordance analysis"
