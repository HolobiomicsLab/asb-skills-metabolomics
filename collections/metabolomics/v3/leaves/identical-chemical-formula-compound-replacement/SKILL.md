---
name: identical-chemical-formula-compound-replacement
description: Use when stress-testing ORA robustness in metabolomics by replacing a
  fraction of compounds with identical-chemical-formula KEGG counterparts to quantify
  how many significant enriched pathways are lost or falsely gained.
when_to_use_negative:
- Input compounds have no KEGG identifiers or no identical-chemical-formula KEGG counterparts
  annotated in at least one KEGG pathway, so valid replacements cannot be drawn
- The goal is to identify exactly which biological pathways are truly affected by
  a perturbation — the authors note experimental ground-truth datasets cannot pinpoint
  the exact affected pathway(s) and simulated ones are too simplistic
- You need multiple-testing-corrected significance calls — this skill's loss/gain
  metric uses a p <= 0.1 SEP threshold without correction, by the study's own design
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test implementing ORA on each perturbed compound
    list to recalculate pathway p-values
  repo: github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Runs the reproducible_simulations.ipynb cells that execute the replacement
    simulation, 100 random samplings, and loss/gain rate aggregation
  repo: github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative to run the same simulations notebook without local
    installation
  repo: https://research.google.com/colaboratory/faq.html
- name: Python 3.8
  role: Language in which all simulations, including the misidentification replacement
    arm, were performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:443ca070203ff607a73ecbfdae5c1e60fdff84aead60b268c50fdebab369085d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - Python 3.8
  - scipy stats fisher_exact
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# identical-chemical-formula-compound-replacement

## Summary

Simulates metabolite misidentification in over-representation analysis (ORA) by replacing a fraction of compounds (e.g. 4%) in the differential metabolite list with KEGG compounds of identical chemical formula, then quantifying the resulting pathway loss and gain rates against the 0% baseline.

## When to use

Use when stress-testing ORA robustness in metabolomics by replacing a fraction of compounds with identical-chemical-formula KEGG counterparts to quantify how many significant enriched pathways are lost or falsely gained.

## When NOT to use

- Input compounds have no KEGG identifiers or no identical-chemical-formula KEGG counterparts annotated in at least one KEGG pathway, so valid replacements cannot be drawn
- The goal is to identify exactly which biological pathways are truly affected by a perturbation — the authors note experimental ground-truth datasets cannot pinpoint the exact affected pathway(s) and simulated ones are too simplistic
- You need multiple-testing-corrected significance calls — this skill's loss/gain metric uses a p <= 0.1 SEP threshold without correction, by the study's own design

## Inputs

- Differential metabolite lists from five MS-based untargeted metabolomics datasets (e.g. MTBLS135, MTBLS136, Yachida et al. 2019 colorectal cancer stool)
- Assay-specific background/reference compound sets
- KEGG pathway sets restricted to pathways containing at least 3 compounds
- Baseline (0% misidentification) list of SEPs significant at p <= 0.1
- Misidentification fraction f (e.g. 4%) and number of replicates (100)

## Outputs

- Per-replicate perturbed compound lists after identical-chemical-formula replacement
- Pathway loss rates and pathway gain rates per replicate
- Per-dataset table of mean and standard error of the mean loss/gain rates for the identical-formula replacement mode

## How to apply

First record the 0% misidentification ORA baseline: run a right-tailed Fisher's exact test (scipy stats fisher_exact) over KEGG pathway sets containing at least 3 compounds, and mark significantly enriched pathways (SEPs) at p <= 0.1. Then randomly select the chosen fraction (e.g. 4%) of compounds and replace each with a KEGG compound of identical chemical formula, restricting replacements to compounds annotated in at least one KEGG pathway and not already part of the original background list, to avoid introducing duplicate compounds. Repeat this randomised replacement 100 times, re-running the ORA on every perturbed compound list, and compute per-replicate pathway loss rate (baseline SEPs no longer significant) and gain rate (newly significant pathways) per Eqs 2-3 of the study. Aggregate over the 100 samplings, reporting mean and standard error of the mean per dataset, and compare against the parallel approximately-isobaric (±20 ppm) replacement arm using the reproducible_simulations.ipynb notebook.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test implementing ORA on each perturbed compound list to recalculate pathway p-values) — github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Runs the reproducible_simulations.ipynb cells that execute the replacement simulation, 100 random samplings, and loss/gain rate aggregation) — github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative to run the same simulations notebook without local installation) — https://research.google.com/colaboratory/faq.html
- **Python 3.8** (Language in which all simulations, including the misidentification replacement arm, were performed)

## Examples

```
git clone https://github.com/cwieder/metabolomics-ORA.git && cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- At 4% misidentification, all five datasets show nonzero pathway loss AND gain rates under the identical-chemical-formula mode, reproducing the Fig 5 finding
- No duplicate compounds are introduced: every replacement compound is annotated in at least one KEGG pathway and absent from the original background list
- Exactly 100 randomised samplings are aggregated and reported as mean plus standard error of the mean per dataset
- The 0% baseline ORA (p <= 0.1, pathways with >= 3 compounds) reproduces the original SEP list before any replacement is applied
- Identical-formula loss/gain rates are of the same order as the parallel ±20 ppm isobaric replacement arm for the same dataset

## Limitations

- No benchmark level of metabolite misidentification typical of metabolomics studies exists, so the 4% rate is illustrative rather than calibrated
- The p <= 0.1 SEP threshold used without multiple testing correction is itself a stated limitation of the loss/gain analysis
- Replacements are restricted to KEGG-annotated compounds with identical chemical formula, so compounds lacking valid formula-matched counterparts cannot be perturbed
- The study is limited by the lack of a ground-truth dataset where the identities of enriched pathways are known, so 'lost' and 'gained' pathways cannot be validated as truly correct or false
- Only pathways with at least 3 compounds are considered, excluding small KEGG pathway sets from the simulation

## Evidence

- [discussion] Simulation of metabolite misidentification by swapping compounds with isobaric (±20 ppm) or identical chemical formula counterparts: [evidence span withheld — anchor status: not_found]
- [discussion] Replacement compounds must be present in at least one KEGG pathway but must not already form part of the original background list, to avoid introducing duplicate compounds.: "Replacement compounds must be present in at least one KEGG pathway but must not already form part of the original background list, to avoid introducing duplicate compounds."
- [discussion] Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean.: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [abstract] Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets.: "Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets."
- [supplementary] (a) identical chemical formula and (b) molecular mass within a +/- 20ppm window: "(a) identical chemical formula and (b) molecular mass within a +/- 20ppm window"
- [readme] Users may adapt the code in the notebook to perform the simulations on their own data.: "Users may adapt the code in the notebook to perform the simulations on their own data."
