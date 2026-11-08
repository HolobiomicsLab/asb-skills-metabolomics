---
name: compound-pathway-size-filtering
description: 'Use when building ORA inputs for metabolomics: after harmonising metabolite
  identifiers to KEGG, Reactome (ChEBI), or BioCyc identifiers, before running the
  right-tailed Fisher''s exact test per pathway.'
when_to_use_negative:
- Metabolite identifiers have not yet been harmonised to the target pathway database
  identifiers (KEGG, ChEBI/Reactome, BioCyc), since the size filter presupposes compound-only
  pathway sets.
- The goal is an extensive functional comparison of pathway databases themselves,
  which the authors explicitly state is beyond the scope of this paper.
- The pathway p-value would be computed for a pathway with 0 overlapping differentially
  abundant metabolites (k = 0); here the rule is to return 1/NA instead of testing.
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scipy stats fisher_exact
  role: Computes the right-tailed (greater) Fisher's exact p-value per pathway after
    the >= 3-compound filter; used as the reference implementation for validating
    the custom ORA module.
  repo: github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: reproducible_simulations.ipynb implements the pathway-set construction, size
    filtering, and ORA simulations.
  repo: github.com/cwieder/metabolomics-ORA
- name: bioservices (Python package) v 1.7.1
  role: Maps KEGG compounds to ChEBI identifiers so Reactome pathway sets can be built
    as compound-only sets prior to filtering.
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:98c1bf5f2a7e978c5b3048ae0e02a1d818449bb9ae6dcd9b662688869f923a51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - Python
  - bioservices (Python package) v 1.7.1
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# compound-pathway-size-filtering

## Summary

Filters the metabolite pathway collection to compound-only pathways containing at least 3 compounds before over-representation analysis, so that Fisher's exact 2x2 contingency tables remain well-posed and trivially small pathways cannot be declared significantly enriched.

## When to use

Use when building ORA inputs for metabolomics: after harmonising metabolite identifiers to KEGG, Reactome (ChEBI), or BioCyc identifiers, before running the right-tailed Fisher's exact test per pathway.

## When NOT to use

- Metabolite identifiers have not yet been harmonised to the target pathway database identifiers (KEGG, ChEBI/Reactome, BioCyc), since the size filter presupposes compound-only pathway sets.
- The goal is an extensive functional comparison of pathway databases themselves, which the authors explicitly state is beyond the scope of this paper.
- The pathway p-value would be computed for a pathway with 0 overlapping differentially abundant metabolites (k = 0); here the rule is to return 1/NA instead of testing.

## Inputs

- Compound-only pathway sets from KEGG, Reactome (ChEBI identifiers), and BioCyc/HumanCyc
- Harmonised metabolite tables per dataset (MTBLS135, Yachida 2019 stool, MTBLS136, Quiros 2017, Fuhrer 2017) with abundance values, class labels, and differential-abundance calls
- List of differentially abundant metabolites (q <= 0.05, BH FDR) and assay-specific background compound set of size N

## Outputs

- Filtered pathway collection restricted to pathways with >= 3 compounds, per database per dataset
- Per-pathway p-values (right-tailed Fisher's exact), with 1/NA for pathways where k < 1
- Per-case validation table (columns N, n, M, k, module p-value, reference p-value, match flag) written to ora_validation_table.csv
- SEP counts per database per dataset (S1 Table A / ora_concordance_table.csv)

## How to apply

Build compound-only pathway sets from the chosen databases (KEGG release 96, Reactome release 75, HumanCyc/BioCyc v24.5 in the paper's concordance simulation) and retain only pathways with at least 3 compounds as ORA input. During the subsequent right-tailed hypergeometric/Fisher's exact scoring (scipy.stats.fisher_exact, alternative='greater'), additionally compute a pathway p-value only when k >= 1 differentially abundant metabolites map to the pathway; otherwise return 1/NA rather than a spuriously small p-value. This mirrors the paper's ORA module, which was validated against scipy's right-tailed Fisher's exact test across a grid of (N, n, M, k) contingency-table cases including M < 3 and k = 0 to exercise the eligibility rules. Keep N (assay-specific background-set size) as an explicit parameter so the filtered pathways are scored against the correct background.

## Related tools

- **scipy stats fisher_exact** (Computes the right-tailed (greater) Fisher's exact p-value per pathway after the >= 3-compound filter; used as the reference implementation for validating the custom ORA module.) — github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (reproducible_simulations.ipynb implements the pathway-set construction, size filtering, and ORA simulations.) — github.com/cwieder/metabolomics-ORA
- **bioservices (Python package) v 1.7.1** (Maps KEGG compounds to ChEBI identifiers so Reactome pathway sets can be built as compound-only sets prior to filtering.)

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Every pathway in the filtered collection has >= 3 compounds; assert this count per database (KEGG, Reactome, BioCyc) and dataset.
- No p-value is emitted for pathways where k = 0; the module returns 1/NA in those grid cases, including explicit M < 3 and k = 0 test cases.
- Module p-values agree with scipy.stats.fisher_exact (alternative='greater') reference values within floating-point tolerance across the entire (N, n, M, k) grid.
- Pathway counts drop after filtering relative to the raw database pathway counts, and the reduction is reported per database.
- Downstream SEP tables (p <= 0.1) list only pathways that passed the size filter.

## Limitations

- The >= 3-compound threshold is a convention used by this study; the paper offers recommendations rather than a benchmarked optimum.
- SEPs in most examples were selected at p <= 0.1 without multiple testing correction, a stated limitation of the study.
- No ground-truth dataset exists where the identities of enriched pathways are known, limiting validation of filter choices against biological truth.
- Code was tested with Python 3.8 on MacOS (v11.2.3) with 16GB RAM; other environments are unverified.

## Evidence

- [discussion] Only pathways containing at least 3 compounds were used as input for ORA.: "Only pathways containing at least 3 compounds were used as input for ORA."
- [discussion] p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: [evidence span withheld — anchor status: not_found]
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
