---
name: isobaric-compound-mass-matching
description: 'Use when assessing the robustness of metabolomics ORA results to metabolite
  misidentification: you have a differential metabolite list, a background compound
  set, and KEGG pathway sets, and need to quantify pathway loss/gain under simulated
  misidentification at a chosen rate (e.g. 4%).'
when_to_use_negative:
- 'You need a true annotation-correction tool: this is an in-silico simulation of
  misidentification impact, not a method for identifying or correcting misidentified
  metabolites.'
- You need to compare pathway databases or benchmark against a ground-truth set of
  enriched pathways — the study explicitly lacks a ground-truth dataset where the
  identities of enriched pathways are known.
- Your dataset is not MS-based untargeted metabolomics with compound identifiers mappable
  to KEGG (the study was limited to five MS-based untargeted metabolomics datasets).
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to recompute pathway p-values on each
    perturbed compound list
- name: Jupyter Notebook
  role: reproducible_simulations.ipynb contains the simulation code; launched locally
    with jupyter-notebook
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative to run reproducible_simulations.ipynb without local
    installation
  repo: https://research.google.com/colaboratory/faq.html
- name: Python 3.8
  role: Language in which all simulations were performed (tested on MacOS v11.2.3,
    16GB RAM)
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:9991f22cfd7354a075b841969c044241531e3882af6dfb513973ceaa8176ff85
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# isobaric-compound-mass-matching

## Summary

Simulates metabolite misidentification by replacing a fraction of compounds in a differential metabolite list with approximately isobaric KEGG compounds (molecular weight within ±20 ppm) or identical-chemical-formula counterparts, then quantifies the resulting pathway loss and gain rates in over-representation analysis (ORA).

## When to use

Use when assessing the robustness of metabolomics ORA results to metabolite misidentification: you have a differential metabolite list, a background compound set, and KEGG pathway sets, and need to quantify pathway loss/gain under simulated misidentification at a chosen rate (e.g. 4%).

## When NOT to use

- You need a true annotation-correction tool: this is an in-silico simulation of misidentification impact, not a method for identifying or correcting misidentified metabolites.
- You need to compare pathway databases or benchmark against a ground-truth set of enriched pathways — the study explicitly lacks a ground-truth dataset where the identities of enriched pathways are known.
- Your dataset is not MS-based untargeted metabolomics with compound identifiers mappable to KEGG (the study was limited to five MS-based untargeted metabolomics datasets).

## Inputs

- Differential metabolite lists (differentially abundant compounds) for each dataset
- Assay-specific background compound sets
- KEGG pathway sets (pathways restricted to at least 3 compounds)
- Compound molecular weights / chemical formulas for KEGG compounds (for ±20 ppm isobaric or identical-formula matching)
- Reproducible_simulations.ipynb notebook (local or Google Colab version)

## Outputs

- Per-dataset pathway loss rate and pathway gain rate at the chosen misidentification rate (e.g. 4%), under both ±20 ppm and identical-formula replacement modes
- Mean and standard error of the mean over 100 random samplings per dataset and mode (Fig 5-style table)

## How to apply

Load the five datasets' differential metabolite lists, background sets, and KEGG pathway sets restricted to pathways containing at least 3 compounds, and record the 0% misidentification baseline of pathways significant at p <= 0.1. Randomly select f% (e.g. 4%) of compounds that have at least one other compound with a molecular weight within ±20 ppm present in the KEGG pathway set and replace them with those approximately isobaric compounds; in a second arm, replace with KEGG compounds of identical chemical formula that are annotated in at least one KEGG pathway and not already part of the original background list (to avoid duplicates). Repeat each randomised replacement 100 times, re-running right-tailed Fisher's exact test ORA (scipy stats fisher_exact) via the reproducible_simulations.ipynb notebook. Compute per-replicate pathway loss rate (proportion of baseline p <= 0.1 pathways no longer significant) and pathway gain rate (proportion of newly significant pathways), then aggregate as mean and standard error of the mean per dataset and per replacement mode.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to recompute pathway p-values on each perturbed compound list)
- **Jupyter Notebook** (reproducible_simulations.ipynb contains the simulation code; launched locally with jupyter-notebook) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative to run reproducible_simulations.ipynb without local installation) — https://research.google.com/colaboratory/faq.html
- **Python 3.8** (Language in which all simulations were performed (tested on MacOS v11.2.3, 16GB RAM)) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- All five datasets should show nonzero pathway loss and gain rates under both the ±20 ppm and identical-formula replacement modes at 4% misidentification, reproducing the Fig 5 finding.
- Each simulation arm must be repeated 100 times with results reported as mean plus standard error of the mean; a single-run result is not acceptable.
- Replacement compounds must be present in at least one KEGG pathway but not already part of the original background list (no duplicate compounds introduced).
- Only pathways containing at least 3 compounds are used as ORA input; p-values are computed only when k (differentially abundant metabolites in the pathway) is >= 1.
- Loss/gain rates must be computed relative to the 0% misidentification baseline of pathways significant at p <= 0.1 without multiple testing correction.

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so loss/gain rates cannot be validated against truth.
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select significantly enriched pathways (SEPs).
- No benchmark level of metabolite misidentification typical of metabolomics studies exists; the 4% rate is a simulation choice, and most studies will contain at least some misidentified compounds.
- The study was limited to five MS-based untargeted metabolomics datasets; two datasets (Quiros et al. 2017 and Fuhrer et al. 2017) lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications.
- Code was tested using Python 3.8 on MacOS (v11.2.3) with 16GB RAM; other environments are untested.

## Evidence

- [discussion] Simulation of metabolite misidentification by swapping compounds with isobaric (±20 ppm) or identical chemical formula counterparts: [evidence span withheld — anchor status: not_found]
- [discussion] Replacement compound constraints: "Replacement compounds must be present in at least one KEGG pathway but must not already form part of the original background list, to avoid introducing duplicate compounds."
- [discussion] Loss/gain rate definition: [evidence span withheld — anchor status: not_found]
- [discussion] Replication protocol: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [discussion] ORA implementation and pathway filter: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [readme] Notebook entry point: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
- [readme] Reproducibility environment: "All code has been tested using Python 3.8 on MacOS (v11.2.3) with standard hardware (16GB RAM)."
- [abstract] Core finding: "Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets."
