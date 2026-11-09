---
name: fisher-exact-contingency-table-implementation
description: Use when performing metabolomics ORA and you need per-pathway p-values
  from a right-tailed Fisher's exact test (hypergeometric) on the pathway collection,
  metabolites-of-interest list, and background compound set.
when_to_use_negative:
- Pathways contain fewer than 3 compounds — the paper explicitly excludes these from
  ORA input.
- The metabolites of interest list has zero overlap (k = 0) with a given pathway —
  no p-value is calculated.
- The goal is functional interpretation of metabolomics datasets without a defined
  background set — the paper shows a non-assay-specific background set yields large
  numbers of false-positive pathways.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: scipy stats fisher_exact
  role: Python function used inside the custom script to calculate the right-tailed
    Fisher's exact p-value for each pathway during ORA.
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Python 3.8
  role: Language in which the ORA p-value calculation was implemented for the simulations.
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Container for the reproducible_simulations.ipynb notebook that runs the ORA
    p-value calculations.
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:b239b583e10d51082c4d463b8bf8965f211c6bf8cb5e071661634d4c5d40a2cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# fisher-exact-contingency-table-implementation

## Summary

Computes per-pathway p-values for over-representation analysis (ORA) in metabolomics using a right-tailed Fisher's exact test based on the hypergeometric distribution, as implemented in the paper's custom Python script.

## When to use

Use when performing metabolomics ORA and you need per-pathway p-values from a right-tailed Fisher's exact test (hypergeometric) on the pathway collection, metabolites-of-interest list, and background compound set.

## When NOT to use

- Pathways contain fewer than 3 compounds — the paper explicitly excludes these from ORA input.
- The metabolites of interest list has zero overlap (k = 0) with a given pathway — no p-value is calculated.
- The goal is functional interpretation of metabolomics datasets without a defined background set — the paper shows a non-assay-specific background set yields large numbers of false-positive pathways.

## Inputs

- Collection of pathways or custom metabolite sets (with compound counts per pathway)
- List of metabolites of interest (e.g. differentially abundant metabolites at q <= 0.05, BH FDR)
- Background or reference set of compounds (size N)
- Per-pathway count k of differentially abundant metabolites

## Outputs

- Per-pathway p-values from the right-tailed Fisher's exact (hypergeometric) test
- Final list of significantly enriched pathways after multiple testing correction

## How to apply

Implement ORA p-value calculation with three essential inputs: a collection of pathways (or custom metabolite sets), a list of metabolites of interest, and a background or reference set of compounds. Restrict the pathway collection to those containing at least 3 compounds before analysis. Calculate the p-value for each pathway using a right-tailed Fisher's exact test based on the hypergeometric distribution, testing whether the count k of differentially abundant metabolites in the pathway is >= 1; if k = 0 no p-value is computed. Adjust resulting p-values with a multiple-testing correction (e.g. Benjamini-Hochberg FDR) to obtain the final list of significantly enriched pathways. In the paper this was implemented via a custom Python script calling scipy.stats.fisher_exact with alternative='greater', within reproducible_simulations.ipynb.

## Related tools

- **scipy stats fisher_exact** (Python function used inside the custom script to calculate the right-tailed Fisher's exact p-value for each pathway during ORA.) — https://github.com/cwieder/metabolomics-ORA
- **Python 3.8** (Language in which the ORA p-value calculation was implemented for the simulations.) — https://github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Container for the reproducible_simulations.ipynb notebook that runs the ORA p-value calculations.) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
cd metabolomics-ORA/src; pip3 install -r requirements.txt; jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Only pathways with at least 3 compounds are passed to ORA; filter the pathway collection first.
- p-value is computed only when k (number of differentially abundant metabolites in the pathway) >= 1; cases with k = 0 should yield no p-value.
- Test is right-tailed (hypergeometric / Fisher exact alternative='greater'), not two-tailed or left-tailed.
- Output p-values must be adjusted (e.g. BH FDR) before selecting the final list of significantly enriched pathways.

## Limitations

- Pathway database choice (KEGG, Reactome, BioCyc) and background set composition profoundly change which pathways appear enriched.
- Simulated metabolite misidentification rates as low as 4% cause gain of false-positive pathways and loss of truly significant pathways, corrupting the test input.
- The paper lacks a ground-truth dataset where enriched pathway identities are known, limiting direct validation of ORA output.
- In most examples a p-value threshold of P <= 0.1 without multiple testing correction was used to select significantly enriched pathways (SEPs), which is a limitation of the study.

## Evidence

- [intro] P-values for each pathway are calculated using a right-tailed Fisher's exact test based on the hypergeometric distribution, followed by multiple testing correction.: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [discussion] ORA input filters and k >= 1 rule: [evidence span withheld — anchor status: not_found]
- [discussion] scipy fisher_exact implementation: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [intro] Three essential inputs: "To perform ORA, three essential inputs are required: a collection of pathways (or custom metabolite sets), a list of metabolites of interest, and a background or reference set of compounds."
- [readme] Repository README: installation and notebook launch: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
