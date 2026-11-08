---
name: metabolite-set-overlap-coefficient-computation
description: Use when ORA has been run on the same metabolomics dataset against two
  or more pathway databases and you need a single number capturing how concordant
  the biological function calls are across databases.
when_to_use_negative:
- You need a quantitative database-content comparison itself (e.g. pathway set sizes,
  compound coverage); the authors state that 'an extensive comparison of pathway databases
  is beyond the scope of this paper'
- A ground-truth list of truly enriched pathways exists; overlap coefficients between
  database ORA outputs cannot identify which (if any) result is correct
- Only one pathway database is used — the overlap coefficient is undefined without
  at least two database-specific SEP pools
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to compute per-pathway ORA p-values
    whose significant pathways (SEPs) define the metabolite pools being compared
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb, which performs the per-database ORA and
    computes the pooled overlap coefficients
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Python 3.8
  role: Implementation language for the ORA simulations and overlap-coefficient computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:5b5fc706cc13439137ce07332e25b72463748f4a27d62499253cb0c9b7fd1585
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-set-overlap-coefficient-computation

## Summary

Quantifies consensus between over-representation analysis (ORA) runs on different pathway databases (KEGG, Reactome, HumanCyc/BioCyc) by pooling the metabolites underlying each database's significantly enriched pathways (SEPs) and computing the overlap coefficient |intersection| / min(|set A|, |set B|), as in Wieder et al. 2021.

## When to use

Use when ORA has been run on the same metabolomics dataset against two or more pathway databases and you need a single number capturing how concordant the biological function calls are across databases.

## When NOT to use

- You need a quantitative database-content comparison itself (e.g. pathway set sizes, compound coverage); the authors state that 'an extensive comparison of pathway databases is beyond the scope of this paper'
- A ground-truth list of truly enriched pathways exists; overlap coefficients between database ORA outputs cannot identify which (if any) result is correct
- Only one pathway database is used — the overlap coefficient is undefined without at least two database-specific SEP pools

## Inputs

- Post-harmonisation metabolite abundance tables with KEGG, ChEBI/Reactome, and BioCyc identifiers, class labels, and differential-abundance calls for each dataset (e.g. MTBLS135, Yachida et al. 2019 stool, MTBLS136, Quiros et al. 2017, Fuhrer et al. 2017)
- Compound-only pathway sets from KEGG (release 96), Reactome (release 75), and HumanCyc (BioCyc v24.5)
- Assay-specific background sets of compounds per dataset
- Lists of differentially abundant metabolites (e.g. selected at q <= 0.05 by BH FDR-adjusted two-tailed t-tests)

## Outputs

- Per-dataset, per-database counts of significantly enriched pathways (SEPs) at p <= 0.1
- Pooled overlap coefficients between KEGG-SEP and Reactome-SEP metabolite pools per dataset (e.g. written to ora_concordance_table.csv)

## How to apply

First run ORA independently per database: build compound-only pathway sets from KEGG (release 96), Reactome (release 75), and HumanCyc (BioCyc v24.5), retain only pathways with at least 3 compounds, and compute per-pathway p-values with the right-tailed scipy.stats.fisher_exact test against the assay-specific background set, calculating a p-value only when k >= 1 differentially abundant metabolites map to the pathway. Select SEPs at p <= 0.1 without multiple testing correction and tabulate SEP counts per database per dataset. Then, for each dataset, pool the metabolites underlying the KEGG SEPs and the metabolites underlying the Reactome SEPs and compute the overlap coefficient as the size of the intersection divided by the size of the smaller pool. Write the SEP counts and pooled overlap coefficients to a table (e.g. ora_concordance_table.csv). In Wieder et al., the five datasets' pooled OC values spanned 0.23 (Stevens/MTBLS135-like) to 0.62 (Labbé), which the authors interpreted as only low to medium consensus between database-specific ORA results.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to compute per-pathway ORA p-values whose significant pathways (SEPs) define the metabolite pools being compared)
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb, which performs the per-database ORA and computes the pooled overlap coefficients) — https://github.com/cwieder/metabolomics-ORA
- **Python 3.8** (Implementation language for the ORA simulations and overlap-coefficient computation)

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- SEP selection invariants hold: every pathway used has at least 3 compounds, and p-values were only calculated when k >= 1 differentially abundant metabolites map to the pathway
- All five datasets yield a finite pooled overlap coefficient in (0, 1], with the published reference range spanning 0.23 (Stevens) to 0.62 (Labbé)
- Overlap coefficient equals |intersection of KEGG-SEP and Reactome-SEP metabolite pools| / min(|KEGG pool|, |Reactome pool|); verify by recomputing from the pooled metabolite lists
- ora_concordance_table.csv contains SEP counts per database per dataset plus exactly five pooled OC values (one per dataset)
- SEPs were selected at p <= 0.1 without multiple testing correction — confirm no BH/Bonferroni step was applied before pooling

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so the overlap coefficient measures consensus, not correctness
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs, which affects which metabolites enter the pools
- The comparison was limited to five MS-based untargeted metabolomics datasets; overlap coefficients may not generalise to other assay types or NMR data

## Evidence

- [results] Overlap coefficients between metabolites pooled from significant pathways detected via KEGG vs Reactome ranged from 0.23 (Stevens) to 0.62 (Labbé): [evidence span withheld — anchor status: not_found]
- [discussion] ORA p-value computation details: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Pathway minimum size filter: [evidence span withheld — anchor status: not_found]
- [abstract] Pathway database choice finding: "Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results in both the number and function of"
- [readme] README notebook usage: "This repository contains the code to run the simulations presented in the study. The Python code to generate the results is contained within the Jupyter notebook"
