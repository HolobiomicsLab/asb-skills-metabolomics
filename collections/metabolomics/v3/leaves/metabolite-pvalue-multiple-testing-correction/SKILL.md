---
name: metabolite-pvalue-multiple-testing-correction
description: Use when metabolite-level p-values (e.g. from two-tailed t-tests per
  compound) must be converted into a filtered list of differentially abundant metabolites
  for over-representation analysis.
when_to_use_negative:
- The input is already a filtered list of metabolite identifiers, not raw metabolite
  p-values — threshold selection is not the step needed.
- Pathway-level p-values from ORA need correction (the paper's SEP selection used
  p ≤ 0.1 without multiple testing correction); this skill targets metabolite-level
  test statistics only.
- Group comparisons are not simple two-group contrasts; the two-tailed Student's t-test
  basis for the p-values would not apply.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Python 3.8
  role: Implementation language for the t-tests, BH FDR and Bonferroni adjustment,
    and sensitivity simulations
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used downstream to convert the corrected
    DA list into pathway p-values
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb, which tabulates SEP counts by correction
    method and α threshold and plots sensitivity curves
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based equivalent of the Jupyter notebook requiring no local installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:6acbed20a66ea40793497af4dc4b8b707eda26ad3caa7dfcdd9516d3ccfb1171
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-pvalue-multiple-testing-correction

## Summary

Adjusts metabolite t-test p-values for multiple testing using Bonferroni or Benjamini-Hochberg FDR procedures to produce the differentially abundant (DA) metabolite list that serves as ORA input, and quantifies how threshold and correction choice alter downstream pathway counts.

## When to use

Use when metabolite-level p-values (e.g. from two-tailed t-tests per compound) must be converted into a filtered list of differentially abundant metabolites for over-representation analysis.

## When NOT to use

- The input is already a filtered list of metabolite identifiers, not raw metabolite p-values — threshold selection is not the step needed.
- Pathway-level p-values from ORA need correction (the paper's SEP selection used p ≤ 0.1 without multiple testing correction); this skill targets metabolite-level test statistics only.
- Group comparisons are not simple two-group contrasts; the two-tailed Student's t-test basis for the p-values would not apply.

## Inputs

- Post-processed metabolite abundance matrix (imputed as half-minimum, log2-transformed, auto-scaled) with condition-group labels
- Vector of raw two-tailed t-test p-values, one per metabolite
- Threshold α for q-value filtering (tested at 0.005, 0.05, 0.1; default q ≤ 0.05)
- Choice of multiple-testing correction: Bonferroni or Benjamini-Hochberg FDR

## Outputs

- Per-metabolite adjusted p-values (q-values) under the chosen correction procedure
- Filtered list of differentially abundant metabolites at the selected q-value threshold
- Counts of significantly enriched pathways (SEPs, p ≤ 0.1) per candidate DA list, tabulated in sep_counts_table.csv
- Sensitivity curves of SEP count versus DA-list construction plotted in reproducible_simulations.ipynb

## How to apply

Compute two-tailed Student's t-tests for every metabolite between condition groups on the post-processed abundance matrix (missing values imputed as half of each metabolite's minimum, log2-transformed, auto-scaled). Adjust the raw p-values with either the Benjamini-Hochberg FDR or Bonferroni procedure; the paper's default DA selection is q ≤ 0.05, but thresholds of α = 0.005, 0.05, and 0.1 were explicitly tested. To probe sensitivity, also construct cumulative candidate lists that add one compound at a time in ascending p-value order. Feed the resulting DA list into right-tailed Fisher's exact ORA and count significantly enriched pathways (SEPs, p ≤ 0.1 without correction) — Bonferroni yields fewer SEPs than BH FDR, and SEP counts rise to a maximum then fall to zero as the DA list approaches the full background set.

## Related tools

- **Python 3.8** (Implementation language for the t-tests, BH FDR and Bonferroni adjustment, and sensitivity simulations)
- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used downstream to convert the corrected DA list into pathway p-values)
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb, which tabulates SEP counts by correction method and α threshold and plots sensitivity curves) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based equivalent of the Jupyter notebook requiring no local installation)

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Bonferroni-adjusted selection yields fewer significant enriched pathways (SEPs) than BH FDR at the same α — the expected ordering Bonferroni < BH FDR.
- SEP counts rise to a global maximum then fall to zero once the DA list equals the entire background set; single compounds can cause local extrema.
- Sensitivity curves for all five datasets show the same rise-then-fall trend when DA metabolites are added in ascending t-test p-value order.
- Default DA selection at q ≤ 0.05 reproduces the paper's baseline for comparison against thresholds 0.005, 0.05, and 0.1.
- Each DA list is non-empty only if at least k ≥ 1 DA metabolite maps to a pathway for a p-value to be computed.

## Limitations

- No benchmark level of metabolite misidentification typical of metabolomics studies exists, so DA-list quality may be degraded by misidentified compounds.
- Two datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications and thus in the p-values fed to the correction.
- A p ≤ 0.1 threshold without multiple testing correction was used to select SEPs in most examples, limiting the interpretability of downstream SEP counts.
- Defining the ideal assay-specific background set remains an open problem, which affects the DA-list-to-background boundary used in the sensitivity analysis.
- Study limited to five MS-based untargeted metabolomics datasets; generalization of threshold sensitivity beyond these is untested.

## Evidence

- [discussion] Differentially abundant metabolites selected at q <= 0.05 (BH FDR); thresholds 0.005, 0.05, 0.1 tested: [evidence span withheld — anchor status: not_found]
- [discussion] Selection of differentially abundant metabolites via two-tailed t-tests with BH FDR / Bonferroni adjustment: [evidence span withheld — anchor status: not_found]
- [discussion] Post-processing steps preceding t-tests on the abundance matrix: "Missing abundance values were imputed using the minimum value of each metabolite divided by 2. All abundance values in the matrix were then log2 transformed and features (metabolites) were auto-scaled"
- [readme] README describing the Jupyter notebook that runs the simulations: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**. Users may adapt the code in the notebook to perform the simulations on their"
