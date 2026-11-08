---
name: hypergeometric-enrichment-pvalue-computation
description: Use when performing ORA on metabolomics data and per-pathway enrichment
  p-values are needed from a pathway collection, a list of metabolites of interest,
  and a background compound set.
when_to_use_negative:
- The input is not a metabolic pathway/metabolite-set enrichment problem — this skill
  computes enrichment p-values, not general hypergeometric draws or GSEA-style rank-based
  scores.
- 'The pathway contains fewer than 3 compounds: the reference workflow states ''Only
  pathways containing at least 3 compounds were used as input for ORA.'''
- 'No metabolites of interest overlap the pathway (k = 0): the reference implementation
  skips p-value calculation in this case.'
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3524
tools:
- name: scipy.stats.fisher_exact
  role: Computes the right-tailed Fisher's exact test p-value (equivalent to the hypergeometric
    upper tail) for each pathway's contingency table
- name: Python 3.8
  role: Implementation environment for the custom ORA script
- name: reproducible_simulations.ipynb
  role: Jupyter notebook containing the ORA p-value computation code users can adapt
    to their own data
  repo: https://github.com/cwieder/metabolomics-ORA.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:105a8a82b6cc45de435b8ac3b6dfbb0225d5037fc3440c6c4802149556317414
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA.git
  tools:
  - Python
  - Python 3.8
  - reproducible_simulations.ipynb
  - scipy.stats.fisher_exact
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# hypergeometric-enrichment-pvalue-computation

## Summary

Computes per-pathway over-representation p-values for metabolomics ORA using a right-tailed Fisher's exact test based on the hypergeometric distribution, as implemented in the metabolomics-ORA reference workflow.

## When to use

Use when performing ORA on metabolomics data and per-pathway enrichment p-values are needed from a pathway collection, a list of metabolites of interest, and a background compound set.

## When NOT to use

- The input is not a metabolic pathway/metabolite-set enrichment problem — this skill computes enrichment p-values, not general hypergeometric draws or GSEA-style rank-based scores.
- The pathway contains fewer than 3 compounds: the reference workflow states 'Only pathways containing at least 3 compounds were used as input for ORA.'
- No metabolites of interest overlap the pathway (k = 0): the reference implementation skips p-value calculation in this case.

## Inputs

- Pathway collection (metabolite sets, filtered to pathways with >= 3 compounds)
- List of metabolites of interest (e.g. differentially abundant metabolites selected at q <= 0.05 BH FDR)
- Background or reference set of compounds (size N)
- Per-pathway overlap count k (differentially abundant metabolites in the pathway)

## Outputs

- Right-tailed Fisher's exact (hypergeometric) p-value per pathway
- Pathways passing the M >= 3 and k >= 1 eligibility gates, ready for multiple-testing correction

## How to apply

Formulate the ORA contingency problem with four parameters: background-set size N, number of metabolites of interest n, number of compounds in the pathway M, and the overlap k of differentially abundant metabolites in that pathway. Compute each pathway's p-value as the upper (right) tail of the hypergeometric distribution — equivalently via a right-tailed Fisher's exact test, which the reference implementation performs with the scipy.stats fisher_exact function (alternative='greater'). Apply the eligibility rules from the reference workflow: only pathways containing at least 3 compounds are used as ORA input, and p-values are only calculated when k >= 1 (the number of differentially abundant metabolites in the ith pathway); pathways failing these gates receive no significant p-value. The p-values feed downstream multiple-testing correction to obtain the final list of significantly enriched pathways. Validate the scoring function by checking agreement against an independent hypergeometric / Fisher's-exact reference across a grid of (N, n, M, k) contingency-table cases within floating-point tolerance.

## Related tools

- **scipy.stats.fisher_exact** (Computes the right-tailed Fisher's exact test p-value (equivalent to the hypergeometric upper tail) for each pathway's contingency table)
- **Python 3.8** (Implementation environment for the custom ORA script)
- **reproducible_simulations.ipynb** (Jupyter notebook containing the ORA p-value computation code users can adapt to their own data) — https://github.com/cwieder/metabolomics-ORA.git

## Examples

```
from scipy.stats import fisher_exact; p = fisher_exact([[k, n-k], [M-k, N-M-n+k]], alternative='greater')[1]  # right-tailed hypergeometric ORA p-value
```

## Evaluation signals

- P-value matches the right-tailed hypergeometric P(X >= k) / Fisher's exact alternative='greater' reference within floating-point tolerance across a grid of (N, n, M, k) contingency cases.
- No p-value is emitted for pathways with fewer than 3 compounds (M < 3 gate).
- No p-value is emitted when the overlap count k = 0 (k >= 1 gate).
- All three ORA inputs (pathway collection, metabolites-of-interest list, background set of size N) were supplied; the background set is explicit, not implicit from the pathway database.
- Downstream multiple-testing correction is applied to the per-pathway p-values before selecting significantly enriched pathways.

## Limitations

- The authors note a limitation: in the majority of examples, a p-value threshold of P <= 0.1 was used without multiple testing correction to select SEPs, so per-pathway p-values may be interpreted without correction.
- There is no ground-truth dataset where the identities of enriched pathways are known, limiting absolute validation of enrichment calls.
- Results depend critically on the background set: using a non-assay-specific background set resulted in large numbers of false-positive pathways.
- Pathway database choice (KEGG, Reactome, BioCyc) led to vastly different results in the number and function of significantly enriched pathways, so p-values are database-specific.
- Code has been tested using Python 3.8 on MacOS (v11.2.3) with 16GB RAM; other environments are untested per the README.

## Evidence

- [intro] P-values for each pathway are calculated using a right-tailed Fisher's exact test based on the hypergeometric distribution.: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA; p-values were calculated if k was >= 1.: [evidence span withheld — anchor status: not_found]
- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed).: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [intro] Three essential inputs are required to perform ORA: a collection of pathways, a list of metabolites of interest, and a background or reference set of compounds.: "To perform ORA, three essential inputs are required: a collection of pathways (or custom metabolite sets), a list of metabolites of interest, and a background or reference set of compounds."
- [readme] README: the Python code to generate the results is contained within the Jupyter notebook src/reproducible_simulations.ipynb; users may adapt it to their own data.: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**. Users may adapt the code in the notebook to perform the simulations on their"
