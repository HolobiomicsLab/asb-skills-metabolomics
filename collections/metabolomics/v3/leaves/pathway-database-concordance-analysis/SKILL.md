---
name: pathway-database-concordance-analysis
description: Use when you need to assess whether ORA conclusions on metabolomics datasets
  are robust to the choice of pathway database, e.g. comparing KEGG vs Reactome vs
  BioCyc significantly enriched pathway (SEP) sets.
when_to_use_negative:
- The goal is to benchmark or compare the content and quality of pathway databases
  themselves — the authors explicitly state an extensive comparison of pathway databases
  is beyond the scope of the paper
- You need a ground-truth assessment of which enriched pathways are correct — the
  authors note no ground-truth dataset exists where the identities of enriched pathways
  are known
- The analysis is gene/RNA-seq ORA rather than metabolomics — the workflow is grounded
  in metabolite identifier harmonisation (KEGG, ChEBI, BioCyc) and MS-based untargeted
  metabolomics datasets
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to compute per-pathway ORA p-values
    for each dataset-database combination
- name: Jupyter Notebook
  role: Runs the reproducible_simulations.ipynb notebook that implements the cross-database
    ORA concordance simulations
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Python 3.8
  role: Language in which all simulations, including the database concordance analysis,
    were performed
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:0e0e18768b3d0fc3fa08232679e2eb76441290373481476438d360fa69367e87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pathway-database-concordance-analysis

## Summary

Quantifies how much over-representation analysis (ORA) results agree when the same metabolomics datasets are analysed against different pathway databases (KEGG, Reactome, BioCyc/HumanCyc), by pooling metabolites underlying significantly enriched pathways and computing overlap coefficients.

## When to use

Use when you need to assess whether ORA conclusions on metabolomics datasets are robust to the choice of pathway database, e.g. comparing KEGG vs Reactome vs BioCyc significantly enriched pathway (SEP) sets.

## When NOT to use

- The goal is to benchmark or compare the content and quality of pathway databases themselves — the authors explicitly state an extensive comparison of pathway databases is beyond the scope of the paper
- You need a ground-truth assessment of which enriched pathways are correct — the authors note no ground-truth dataset exists where the identities of enriched pathways are known
- The analysis is gene/RNA-seq ORA rather than metabolomics — the workflow is grounded in metabolite identifier harmonisation (KEGG, ChEBI, BioCyc) and MS-based untargeted metabolomics datasets

## Inputs

- Post-harmonisation metabolite abundance tables per dataset with KEGG, ChEBI/Reactome, and BioCyc compound identifiers, class labels, and differential-abundance calls
- Compound-only pathway sets from KEGG (release 96), Reactome (release 75), and HumanCyc (BioCyc v24.5)
- Assay-specific background compound sets for each dataset

## Outputs

- Per-dataset, per-database counts of significantly enriched pathways (SEP counts, e.g. S1 Table A)
- Pooled overlap coefficients between KEGG-SEP and Reactome-SEP metabolites per dataset (ora_concordance_table.csv)

## How to apply

Load the post-harmonisation metabolite tables for each dataset (KEGG, ChEBI/Reactome, and BioCyc identifiers with abundance values, class labels, and differential-abundance calls). Build compound-only pathway sets from KEGG (release 96), Reactome (release 75), and HumanCyc (BioCyc v24.5), retaining only pathways with at least 3 compounds. For each dataset-database combination, compute per-pathway p-values with the right-tailed scipy.stats.fisher_exact test against the assay-specific background set, calculating a p-value only when k >= 1 differentially abundant metabolites map to the pathway. Select significantly enriched pathways (SEPs) at p <= 0.1 without multiple testing correction, pool the metabolites underlying KEGG SEPs and Reactome SEPs per dataset, and compute the overlap coefficient |intersection| / min(|KEGG pool|, |Reactome pool|). In this study the five datasets yielded pooled OC values spanning 0.23 (Stevens/MTBLS135) to 0.62 (Labbé), interpreted as only low to medium consensus.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to compute per-pathway ORA p-values for each dataset-database combination)
- **Jupyter Notebook** (Runs the reproducible_simulations.ipynb notebook that implements the cross-database ORA concordance simulations) — https://github.com/cwieder/metabolomics-ORA
- **Python 3.8** (Language in which all simulations, including the database concordance analysis, were performed) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- The output ora_concordance_table.csv contains five pooled overlap coefficients spanning 0.23 (Stevens) to 0.62 (Labbé)
- Only pathways containing at least 3 compounds were used as ORA input for every database
- P-values are computed only when k >= 1 differentially abundant metabolites map to a pathway, using the right-tailed Fisher's exact test against an assay-specific background set
- SEPs are selected at p <= 0.1 without multiple testing correction, matching the paper's SEP definition
- SEP counts are tabulated per database per dataset for all five datasets (MTBLS135, Yachida 2019 stool, MTBLS136, Quiros 2017, Fuhrer 2017)

## Limitations

- Only low to medium consensus (OC 0.23–0.62) between databases was observed, so concordance analysis cannot establish which database's results are 'correct'
- SEPs were selected at p <= 0.1 without multiple testing correction, a limitation the authors acknowledge
- Analysis limited to five MS-based untargeted metabolomics datasets, two of which (Quiros 2017, Fuhrer 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications
- No ground-truth dataset exists where the identities of enriched pathways are known, limiting validation of concordance results

## Evidence

- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: "Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites"
- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values.: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [abstract] Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results in both the number and function of: "Pathway database choice, evaluated using three of the most popular metabolic pathway databases (KEGG, Reactome, and BioCyc), led to vastly different results"
- [readme] This repository contains the code to run the simulations presented in the study. The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**.: "The Python code to generate the results 
is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
- [discussion] The pathway loss rate represents the proportion of the original pathways (0% misidentification) significant at p <= 0.1 that are no longer significant at f % misidentification.: [evidence span withheld — anchor status: not_found]
