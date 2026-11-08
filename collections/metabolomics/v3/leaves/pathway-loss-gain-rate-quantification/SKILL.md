---
name: pathway-loss-gain-rate-quantification
description: Use when you need to stress-test the stability of an ORA-derived list
  of significantly enriched metabolomics pathways against compound annotation errors,
  e.g. measuring how many baseline pathways (p <= 0.1) are lost or falsely gained
  at a given simulated misidentification rate.
when_to_use_negative:
- The goal is a comparison of pathway databases (KEGG vs Reactome vs BioCyc) — the
  authors state an extensive comparison of pathway databases is beyond the scope of
  this work
- The task is identifying truly affected pathways in a real perturbation experiment
  — the study notes no ground-truth dataset exists where the identities of enriched
  pathways are known, and this skill only measures stability under simulated annotation
  error
- The pathway significance list was selected with multiple-testing correction (e.g.
  q <= 0.05 BH FDR) — this skill's loss/gain metric is defined on SEPs at p <= 0.1
  without correction
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2269
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to recompute pathway p-values on every
    perturbed compound list
- name: reproducible_simulations.ipynb (Jupyter Notebook)
  role: Notebook in github.com/cwieder/metabolomics-ORA/src containing the Python
    3.8 code that runs the misidentification simulations, ORA, and loss/gain rate
    aggregation
  repo: github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based execution of the same simulation notebook without local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:4417cf4fea18d15227fad8d9517d68a7ddc0905808886d5eb673c8629afeb7c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - reproducible_simulations.ipynb (Jupyter Notebook)
  - scipy stats fisher_exact
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# pathway-loss-gain-rate-quantification

## Summary

Quantifies the sensitivity of over-representation analysis (ORA) to metabolite misidentification by simulating replacement of compounds with approximately isobaric (±20 ppm) or identical-chemical-formula counterparts and computing pathway loss and gain rates against a 0% misidentification baseline.

## When to use

Use when you need to stress-test the stability of an ORA-derived list of significantly enriched metabolomics pathways against compound annotation errors, e.g. measuring how many baseline pathways (p <= 0.1) are lost or falsely gained at a given simulated misidentification rate.

## When NOT to use

- The goal is a comparison of pathway databases (KEGG vs Reactome vs BioCyc) — the authors state an extensive comparison of pathway databases is beyond the scope of this work
- The task is identifying truly affected pathways in a real perturbation experiment — the study notes no ground-truth dataset exists where the identities of enriched pathways are known, and this skill only measures stability under simulated annotation error
- The pathway significance list was selected with multiple-testing correction (e.g. q <= 0.05 BH FDR) — this skill's loss/gain metric is defined on SEPs at p <= 0.1 without correction

## Inputs

- Five experimental metabolomics datasets with differential metabolite lists, assay-specific background compound sets, and KEGG pathway sets (pathways with >= 3 compounds), e.g. MTBLS135, MTBLS136, Yachida et al. 2019 colorectal cancer stool, Quiros et al. 2017, Fuhrer et al. 2017
- 0% misidentification ORA baseline list of significantly enriched pathways (p <= 0.1)
- KEGG compound records with molecular weights and chemical formulas, mapped via MetaboAnalyst 4.0 ID conversion and bioservices v 1.7.1

## Outputs

- Per-dataset table of pathway loss rate and pathway gain rate (mean ± standard error over 100 random samplings) at 4% misidentification, for both isobaric (±20 ppm) and identical-chemical-formula replacement modes
- Per-replicate lists of lost (no longer significant at p <= 0.1) and gained (newly significant) pathways computed via Eqs 2-3

## How to apply

For each dataset, record the 0% misidentification ORA baseline: run a right-tailed Fisher's exact test (scipy stats fisher_exact) over KEGG pathway sets restricted to pathways containing at least 3 compounds and mark pathways significant at p <= 0.1 (no multiple testing correction). Randomly select 4% of compounds that have another compound within ±20 ppm molecular weight in the KEGG pathway set and replace them with those approximately isobaric compounds; in a second arm, replace them with KEGG compounds of identical chemical formula that are annotated in at least one KEGG pathway and are not already in the background list (to avoid duplicate compounds). Repeat each randomised replacement 100 times, re-running ORA on every perturbed compound list, then apply Eqs 2-3 to compute per-replicate pathway loss rate (baseline pathways no longer significant) and gain rate (newly significant pathways). Aggregate as mean ± standard error of the mean per dataset and per replacement mode; validation is a table showing nonzero loss and gain rates for all five datasets under both modes, reproducing the paper's Fig 5.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to recompute pathway p-values on every perturbed compound list)
- **reproducible_simulations.ipynb (Jupyter Notebook)** (Notebook in github.com/cwieder/metabolomics-ORA/src containing the Python 3.8 code that runs the misidentification simulations, ORA, and loss/gain rate aggregation) — github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based execution of the same simulation notebook without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- All five datasets show nonzero pathway loss and gain rates under both the ±20 ppm isobaric and identical-chemical-formula replacement modes, reproducing the Fig 5 finding
- The 0% misidentification baseline is recorded first and the loss rate is computed relative to pathways significant at p <= 0.1 in that baseline
- Results are reported as the mean of 100 random samplings with the standard error of the mean, not single-run values
- Only pathways containing at least 3 compounds enter ORA, and replacement compounds are in at least one KEGG pathway and absent from the original background list
- ORA p-values are computed with the right-tailed scipy stats fisher_exact, matching the hypergeometric formulation

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so 'gained' pathways can only be flagged as potential false positives, not verified
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select significantly enriched pathways, inflating sensitivity of the metric
- There is no benchmark level of metabolite misidentification typical of metabolomics studies to anchor the chosen 4% rate; two of the five datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications

## Evidence

- [abstract] Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets.: "Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets."
- [discussion] The pathway loss rate represents the proportion of the original pathways (0% misidentification) significant at p <= 0.1 that are no longer significant at f % misidentification.: [evidence span withheld — anchor status: not_found]
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA.: "Only pathways containing at least 3 compounds were used as input for ORA."
- [supplementary] (a) identical chemical formula and (b) molecular mass within a +/- 20ppm window: "(a) identical chemical formula and (b) molecular mass within a +/- 20ppm window"
- [readme] The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**.: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
