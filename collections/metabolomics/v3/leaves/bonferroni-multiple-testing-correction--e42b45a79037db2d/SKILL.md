---
name: bonferroni-multiple-testing-correction
description: Use when a benchmarking workflow produces one P value per metabolite
  per feature set (e.g. Pearson correlations from 2,025 cross-validated RF models)
  and a per-metabolite significance call is needed.
when_to_use_negative:
- When only a handful of metabolites or hypotheses are tested — the 0.05/225 correction
  is calibrated to this panel size and would be overly conservative elsewhere; recompute
  the denominator from the actual number of tests
- When the goal is FDR control at a target q-value rather than family-wise error control
  — a Benjamini-Hochberg procedure would be the appropriate alternative
- When P values are not independent or not validly computed (e.g. correlations from
  overlapping CV folds) — Bonferroni does not repair invalid base tests
edam_operation: http://edamontology.org/operation_2263
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Trains the RandomForestRegressor models whose 5-fold CV Pearson correlations
    and P values are the input to the Bonferroni correction
  repo: sriram-lab/Metab8D
- name: Python
  role: Implements the threshold computation and significance filtering in the Metab8D
    workflow scripts
  repo: sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:dbe802280983645aa2b0668a72575c81344f3301780964b0b676f32f3d55ecbb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - RDKit
  - SciPy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2263
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bonferroni-multiple-testing-correction

## Summary

Determines which metabolite prediction models are statistically significant by applying a Bonferroni-corrected P value threshold of 0.05/225 across the 225 metabolite x 9 feature set models, controlling the false discovery rate when thousands of correlation tests are run in parallel.

## When to use

Use when a benchmarking workflow produces one P value per metabolite per feature set (e.g. Pearson correlations from 2,025 cross-validated RF models) and a per-metabolite significance call is needed.

## When NOT to use

- When only a handful of metabolites or hypotheses are tested — the 0.05/225 correction is calibrated to this panel size and would be overly conservative elsewhere; recompute the denominator from the actual number of tests
- When the goal is FDR control at a target q-value rather than family-wise error control — a Benjamini-Hochberg procedure would be the appropriate alternative
- When P values are not independent or not validly computed (e.g. correlations from overlapping CV folds) — Bonferroni does not repair invalid base tests

## Inputs

- Matrix of Pearson's correlation P values for all 225 metabolite models per feature set (RF_results.csv columns: correlations, P values)
- Number of hypotheses tested (225 metabolites per feature set)

## Outputs

- Bonferroni-corrected P value threshold (0.05/225 ≈ 2.22e-04)
- Per-metabolite binary significance calls for each of the 9 feature sets
- Significance columns embedded in RF_results.csv / Metab8D_network.xlsx alongside correlations and confidence scores

## How to apply

After computing cross-validated Pearson's correlations between predicted and true metabolite concentrations for each of the 225 metabolites under each of the 9 omics feature sets, compute the family-wise Bonferroni threshold as 0.05/225 (the denominator equals the number of metabolites tested per feature set). Label a metabolite model statistically significant only if its P value falls below this corrected threshold. Store the raw P values, correlations, and Bonferroni-based significance calls together (as in RF_results.csv) so downstream users can re-derive significance under alternative corrections. Rationale: with 225 tests per feature set (2,025 total), uncorrected P < 0.05 would inflate false positives; Bonferroni is the correction the article applies to account for false discovery rate.

## Related tools

- **scikit-learn** (Trains the RandomForestRegressor models whose 5-fold CV Pearson correlations and P values are the input to the Bonferroni correction) — sriram-lab/Metab8D
- **Python** (Implements the threshold computation and significance filtering in the Metab8D workflow scripts) — sriram-lab/Metab8D

## Examples

```
import numpy as np; alpha = 0.05/225; sig = np.array(pvals) < alpha  # Bonferroni-corrected significance for 225 metabolite models
```

## Evaluation signals

- The applied threshold must equal 0.05/225 (~2.22e-04) for the 225-metabolite panel; any other denominator indicates the wrong family size
- Every metabolite called significant must have a raw P value strictly below the corrected threshold, and no P value above the threshold may be flagged
- RF_results.csv must contain correlations, P values, and Bonferroni-based significance calls for all 225 metabolites x 9 feature sets (2,025 rows/tests)
- The number of significant metabolites should be substantially smaller than 225 (roughly 20x fewer than an uncorrected P < 0.05 call would yield under the null); a large discrepancy warrants re-checking

## Limitations

- The metabolite panel of 225 metabolites is only a fraction of metabolites that exist within a cell across compartments, so significance calls cover only the measured panel
- Bonferroni is conservative; metabolites with moderate but real predictability may fail the 0.05/225 threshold
- Significance of the correlation does not imply a mechanistic relationship — the 'guilt by association' strategy may not identify causative mechanisms

## Evidence

- [discussion] Bonferroni threshold: "Statistically significant predictions were determined using a Bonferroni-corrected P value (0.05/225) to account for false discovery rate"
- [readme] RF_results description: "RF_results: Pearson's correlations and P values for all metabolite models from each of nine feature sets. Significance is determined by Bonferroni-corrected P value."
- [discussion] performance assessment: [evidence span withheld — anchor status: not_found]
