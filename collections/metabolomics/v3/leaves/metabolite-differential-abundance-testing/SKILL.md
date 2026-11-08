---
name: metabolite-differential-abundance-testing
description: Use when a metabolomics abundance matrix (imputed, log2-transformed,
  auto-scaled) must be reduced to a list of DA metabolites via statistical testing
  before over-representation analysis.
when_to_use_negative:
- The analysis goal is pathway-level enrichment rather than compound-level DA testing
  — feed the DA list into ORA (right-tailed Fisher's exact test, scipy.stats.fisher_exact)
  after selection, not instead of it
- The dataset lacks a two-group contrast structure; this skill was validated on datasets
  such as MTBLS135 (12 Hi-Myc vs 12 wild-type Mus musculus tissue samples) and Yachida
  et al. 2019 (149 healthy control vs 148 colorectal cancer stool samples) with clear
  binary groupings
- Raw, un-normalized abundance data is supplied without prior imputation, log2 transformation,
  and auto-scaling — the t-test results depend on these preprocessing steps
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: Python 3.8
  role: Runtime for implementing the two-tailed Student's t-tests and BH FDR adjustment
    on the abundance matrix within the reproducible_simulations.ipynb Jupyter notebook
  repo: https://github.com/cwieder/metabolomics-ORA
- name: scipy stats fisher_exact
  role: Right-tailed hypergeometric ORA applied to the resulting DA metabolite list
    to count significantly enriched pathways
- name: Jupyter Notebook
  role: Reproducible_simulations.ipynb contains the code cells implementing the t-tests
    and DA metabolite selection across the five datasets
- name: Google Colaboratory
  role: Browser-based alternative for running the notebook without local installation
    of packages or dependencies
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:15673239486934bc3dc20cfbdd04f1e4e28a890cc08ed7ee12d33a1805fa631e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - Python 3.8
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6190476190476191
n_claims: 21
---

# metabolite-differential-abundance-testing

## Summary

Runs two-tailed Student's t-tests on log2-transformed, auto-scaled metabolite abundance matrices to define the list of differentially abundant (DA) metabolites that serves as input to ORA; a key sensitivity parameter for pathway analysis outcomes.

## When to use

Use when a metabolomics abundance matrix (imputed, log2-transformed, auto-scaled) must be reduced to a list of DA metabolites via statistical testing before over-representation analysis.

## When NOT to use

- The analysis goal is pathway-level enrichment rather than compound-level DA testing — feed the DA list into ORA (right-tailed Fisher's exact test, scipy.stats.fisher_exact) after selection, not instead of it
- The dataset lacks a two-group contrast structure; this skill was validated on datasets such as MTBLS135 (12 Hi-Myc vs 12 wild-type Mus musculus tissue samples) and Yachida et al. 2019 (149 healthy control vs 148 colorectal cancer stool samples) with clear binary groupings
- Raw, un-normalized abundance data is supplied without prior imputation, log2 transformation, and auto-scaling — the t-test results depend on these preprocessing steps

## Inputs

- Processed metabolite abundance matrix (missing values imputed, log2-transformed, auto-scaled) with condition group labels
- Choice of multiple-testing correction: Benjamini-Hochberg FDR or Bonferroni
- q-value threshold (default q ≤ 0.05; also tested at 0.005, 0.05, 0.1)

## Outputs

- List of differentially abundant metabolites per dataset
- Cumulative DA metabolite lists ordered by ascending t-test p-value for sensitivity analysis
- Per-compound t-test p-values and BH FDR / Bonferroni-adjusted q-values

## How to apply

Post-process the abundance matrix first: impute missing values as half of each metabolite's minimum, log2-transform, and auto-scale features. For every metabolite compute a two-tailed Student's t-test between condition groups, then select DA metabolites using the Benjamini-Hochberg FDR procedure at a q ≤ 0.05 threshold (alternatively Bonferroni; thresholds of 0.005, 0.05, and 0.1 were evaluated). Do not apply a multiple-testing correction to the resulting pathway p-values; the article used p ≤ 0.1 without correction to count significantly enriched pathways (SEPs). Note that the DA-list construction strongly drives ORA output: adding DA compounds one at a time in ascending t-test p-value order shows SEP counts rising to a global maximum then falling to zero when the list equals the background set.

## Related tools

- **Python 3.8** (Runtime for implementing the two-tailed Student's t-tests and BH FDR adjustment on the abundance matrix within the reproducible_simulations.ipynb Jupyter notebook) — https://github.com/cwieder/metabolomics-ORA
- **scipy stats fisher_exact** (Right-tailed hypergeometric ORA applied to the resulting DA metabolite list to count significantly enriched pathways)
- **Jupyter Notebook** (Reproducible_simulations.ipynb contains the code cells implementing the t-tests and DA metabolite selection across the five datasets)
- **Google Colaboratory** (Browser-based alternative for running the notebook without local installation of packages or dependencies)

## Examples

```
cd metabolomics-ORA/src; pip3 install -r requirements.txt; jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Confirm every metabolite has a computed two-tailed Student's t-test p-value and a BH FDR-adjusted q-value before thresholding
- Verify that only metabolites with q ≤ 0.05 (or the chosen threshold) appear in the DA list used as ORA input
- Reproducing the sensitivity analysis should show the number of significantly enriched pathways (p ≤ 0.1, no correction) rising to a global maximum then falling to zero as the DA list grows toward the full background set
- Bonferroni-adjusted SEP counts should be ≤ BH FDR-adjusted counts at the same α threshold, with the BH FDR maximum occurring at α ≤ 0.05
- Group labels and dataset processing (imputation as half minimum, log2 transform, auto-scaling) must be recorded in sep_counts_table.csv for traceability

## Limitations

- Defining the ideal assay-specific background set for a dataset remains an open problem; the DA list alone does not resolve this
- In most examples a p-value threshold of P ≤ 0.1 was used without multiple testing correction to select SEPs, which is a limitation of the ORA validation approach
- No ground-truth dataset exists where the identities of enriched pathways are known, so DA-test-driven ORA results cannot be checked against true pathway labels
- Only pathways containing at least 3 compounds were used as ORA input; pathway databases with fewer compounds per pathway will yield fewer testable pathways
- Two datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications downstream of the DA testing

## Evidence

- [methods] Differentially abundant metabolites selected at q <= 0.05 (BH FDR); thresholds 0.005, 0.05, 0.1 tested: [evidence span withheld — anchor status: not_found]
- [methods] Missing abundance values were imputed using the minimum value of each metabolite divided by 2. All abundance values in the matrix were then log2 transformed and features (metabolites) were auto-scaled: "Missing abundance values were imputed using the minimum value of each metabolite divided by 2. All abundance values in the matrix were then log2 transformed and features (metabolites) were auto-scaled"
- [methods] Two-tailed student's t-tests with BH FDR adjustment define the metabolites of interest before ORA: [evidence span withheld — anchor status: not_found]
- [abstract] Profoundly different ORA results arise from changing DA metabolite selection methods: "changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results"
- [results] Adding DA metabolites one at a time drives SEP counts to a maximum then zero: [evidence span withheld — anchor status: not_found]
- [methods] Only pathways containing at least 3 compounds were used as input for ORA: "Only pathways containing at least 3 compounds were used as input for ORA."
- [readme] Launching the reproducible simulations notebook to run the DA-testing workflow: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
