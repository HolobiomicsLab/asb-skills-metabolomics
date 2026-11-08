---
name: metabolite-misidentification-simulation
description: Use when assessing the robustness of metabolomics ORA to metabolite misidentification
  — e.g. reproducing pathway loss/gain rates at a fixed misidentification rate (4%)
  across datasets with KEGG-annotated differential metabolites and background lists.
when_to_use_negative:
- The goal is standard ORA on unperturbed data — this skill is a robustness simulation,
  not the primary pathway-enrichment analysis.
- Compounds lack KEGG annotations, so few or no ±20 ppm isobaric or identical-formula
  replacement candidates exist in the pathway set.
- A ground-truth validation of truly enriched pathways is required — the article notes
  no ground-truth dataset with known enriched pathway identities was available.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to recalculate pathway p-values on every
    perturbed compound list per replicate
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Runs the reproducible_simulations.ipynb code cells implementing the misidentification
    simulations locally (Python 3.8)
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same simulations notebook without
    local installation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:f93c058122f05f010d51d04311fcd26b06ee849e5bbe7904c0b53cf239e99485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - Python
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 24
---

# metabolite-misidentification-simulation

## Summary

In-silico simulation that swaps 4% of compounds in a differential metabolite list with isobaric (±20 ppm) or identical-chemical-formula KEGG counterparts, then re-runs right-tailed Fisher's exact test ORA to quantify pathway loss and gain rates versus the 0% misidentification baseline.

## When to use

Use when assessing the robustness of metabolomics ORA to metabolite misidentification — e.g. reproducing pathway loss/gain rates at a fixed misidentification rate (4%) across datasets with KEGG-annotated differential metabolites and background lists.

## When NOT to use

- The goal is standard ORA on unperturbed data — this skill is a robustness simulation, not the primary pathway-enrichment analysis.
- Compounds lack KEGG annotations, so few or no ±20 ppm isobaric or identical-formula replacement candidates exist in the pathway set.
- A ground-truth validation of truly enriched pathways is required — the article notes no ground-truth dataset with known enriched pathway identities was available.

## Inputs

- Differential metabolite lists from five experimental metabolomics datasets (e.g. MTBLS135, MTBLS136, Yachida et al. 2019 colorectal cancer stool dataset)
- Assay-specific background compound sets
- KEGG pathway sets (only pathways containing at least 3 compounds)
- Misidentification rate f (e.g. 4%) and replacement mode (±20 ppm isobaric or identical chemical formula)
- Molecular weights / chemical formulas of KEGG compounds for matching isobaric or formula-identical counterparts

## Outputs

- Per-dataset, per-replacement-mode pathway loss rate (proportion of baseline pathways significant at p <= 0.1 no longer significant)
- Per-dataset, per-replacement-mode pathway gain rate (proportion of newly significant pathways)
- Mean and standard error of the mean aggregated over 100 random samplings, assembled as a per-dataset table

## How to apply

First compute the 0% misidentification baseline: run right-tailed Fisher's exact test ORA (scipy stats fisher_exact) using the differential metabolite list, assay-specific background set, and KEGG pathway sets restricted to pathways containing at least 3 compounds, recording pathways significant at p <= 0.1. Then randomly select 4% of compounds that have at least one other compound with molecular weight within ±20 ppm (approximately isobaric) present in the KEGG pathway set, or — in a second arm — compounds of identical chemical formula; replacement compounds must be annotated in at least one KEGG pathway and must not already be in the original background list (to avoid duplicate compounds). Repeat each randomised replacement 100 times, re-running ORA per replicate via the reproducible_simulations.ipynb notebook in Python (or its Google Colab version). Per replicate, compute the pathway loss rate (baseline pathways at p <= 0.1 no longer significant) and gain rate (newly significant pathways) using Eqs 2-3, then report mean and standard error of the mean per dataset and replacement mode.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to recalculate pathway p-values on every perturbed compound list per replicate) — https://github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Runs the reproducible_simulations.ipynb code cells implementing the misidentification simulations locally (Python 3.8)) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same simulations notebook without local installation)

## Examples

```
cd metabolomics-ORA/src; pip3 install -r requirements.txt; jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- All five datasets show nonzero pathway loss AND gain rates under both ±20 ppm and identical-chemical-formula replacement modes at 4% misidentification, reproducing the Fig 5 finding.
- Each simulation arm comprises 100 random samplings, with results reported as mean plus standard error of the mean.
- Baseline set of significantly enriched pathways (0% misidentification) is defined at p <= 0.1 without multiple testing correction, and loss/gain rates are computed relative to this baseline via Eqs 2-3.
- Replacement compounds are annotated in at least one KEGG pathway and never already part of the original background list (no duplicate compounds introduced).
- Only KEGG pathways containing at least 3 compounds are used as ORA input in every replicate.

## Limitations

- No benchmark level of metabolite misidentification typical of metabolomics studies exists, so the 4% rate is illustrative rather than calibrated.
- The p <= 0.1 threshold for selecting significantly enriched pathways was used without multiple testing correction, a stated limitation of the study.
- Results derive from only five MS-based untargeted metabolomics datasets; two (Quiros et al. 2017, Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications.
- No ground-truth dataset with known identities of enriched pathways was available to validate perturbed results.

## Evidence

- [abstract] abstract finding: "Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets."
- [discussion] isobaric selection: [evidence span withheld — anchor status: not_found]
- [discussion] replacement constraints: "Replacement compounds must be present in at least one KEGG pathway but must not already form part of the original background list, to avoid introducing duplicate compounds."
- [discussion] loss rate definition: [evidence span withheld — anchor status: not_found]
- [discussion] replication scheme: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [readme] notebook location: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
