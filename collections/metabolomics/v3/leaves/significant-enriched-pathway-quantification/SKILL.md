---
name: significant-enriched-pathway-quantification
description: Use when running ORA perturbation simulations (background-set reduction,
  metabolite misidentification, DA-list changes) and you need a robust metric of how
  many enriched pathways are gained or lost across datasets.
when_to_use_negative:
- You need a statistically calibrated significance call for a single analysis — p
  <= 0.1 without multiple-testing correction is only for sensitivity comparison, not
  reporting final enrichment results
- You need ground-truth-validated pathway enrichment — the authors note no dataset
  exists where the identities of enriched pathways are known, so SEP counts measure
  sensitivity, not accuracy
- The input is an unprocessed abundance matrix — DA metabolites must first be derived
  via two-tailed t-tests with BH FDR q <= 0.05 before SEP counting
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact (hypergeometric) test used to compute per-pathway
    p-values whose counts at p <= 0.1 form the SEP metric
- name: Jupyter Notebook
  role: Runs the reproducible_simulations.ipynb cells that execute the ORA simulations
    and tabulate SEP proportions
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same SEP-quantification notebook
    without local installation
- name: Python 3.8
  role: Language in which all SEP-counting simulations were implemented and tested
    (MacOS v11.2.3, 16GB RAM)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:724aec0aadb928450364500f7242db578d5830dcd8e9b1b4c3decd037e347a4e
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# significant-enriched-pathway-quantification

## Summary

Quantifies significantly enriched pathways (SEPs) from over-representation analysis by counting pathways passing p <= 0.1 (without multiple testing correction) after each perturbation, expressed as a retained proportion relative to the unperturbed run, with mean and standard error over 100 randomised repetitions.

## When to use

Use when running ORA perturbation simulations (background-set reduction, metabolite misidentification, DA-list changes) and you need a robust metric of how many enriched pathways are gained or lost across datasets.

## When NOT to use

- You need a statistically calibrated significance call for a single analysis — p <= 0.1 without multiple-testing correction is only for sensitivity comparison, not reporting final enrichment results
- You need ground-truth-validated pathway enrichment — the authors note no dataset exists where the identities of enriched pathways are known, so SEP counts measure sensitivity, not accuracy
- The input is an unprocessed abundance matrix — DA metabolites must first be derived via two-tailed t-tests with BH FDR q <= 0.05 before SEP counting

## Inputs

- OR A p-value matrix (one row per pathway, per dataset) computed with right-tailed Fisher's exact test on k >= 1
- Unperturbed (0%) baseline SEP list per dataset, pathway database (KEGG/Reactome/BioCyc), and DA-list definition
- Perturbed run outputs (reduced background sets down to 10% of original compound count, or misidentification rate f%)
- Simulation replicate index for randomised runs (100 repeats per condition)

## Outputs

- SEP counts and retained-SEP proportions per dataset, removal mode, and reduction step (e.g. background_reduction_sweep.csv)
- Mean SEP proportion with standard error of the mean across 100 repetitions
- Pathway loss/gain rates comparing perturbed runs to the 0% baseline at p <= 0.1

## How to apply

For each ORA run, compute right-tailed Fisher's exact test (hypergeometric) p-values with scipy.stats.fisher_exact, calculating p-values only when k (differentially abundant metabolites in pathway i) >= 1 and restricting to pathways with at least 3 compounds. Count SEPs at p <= 0.1 without multiple-testing correction — a deliberately permissive threshold so gain/loss effects are measurable. Define the SEP proportion as the count significant at a given perturbation level divided by the unperturbed (0%) count, so 'pathway loss rate' is the proportion of original SEPs no longer significant. Repeat every randomised simulation (e.g. random compound removal, misidentification swaps) 100 times and report the mean plus standard error of the mean for every dataset, perturbation mode, and reduction step. Write per-dataset tables/curves (e.g. background_reduction_sweep.csv) and check expected patterns such as SEP proportion declining as the background set shrinks.

## Related tools

- **scipy stats fisher_exact** (Right-tailed Fisher's exact (hypergeometric) test used to compute per-pathway p-values whose counts at p <= 0.1 form the SEP metric)
- **Jupyter Notebook** (Runs the reproducible_simulations.ipynb cells that execute the ORA simulations and tabulate SEP proportions) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same SEP-quantification notebook without local installation)
- **Python 3.8** (Language in which all SEP-counting simulations were implemented and tested (MacOS v11.2.3, 16GB RAM))

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- With no perturbation (0%), the SEP proportion must equal 1.0 for every dataset and pathway database
- Randomised simulations must be run exactly 100 times, with both mean and standard error of the mean reported per condition
- SEP p-values must be computed only for pathways with >= 3 compounds and when k >= 1
- For background-reduction sweeps, the retained SEP proportion should decline as the background shrinks toward 10% of the original count
- Per-dataset output tables/curves must be complete for every dataset, removal mode, and reduction step

## Limitations

- p <= 0.1 without multiple testing correction inflates SEP counts; this threshold is a sensitivity metric, not a corrected significance call
- No ground-truth dataset exists where the identities of enriched pathways are known, so SEP gain/loss cannot be validated against truth
- Quantification depends on upstream choices — background set, DA-metabolite selection, and pathway database — each of which alone produces profoundly different ORA results

## Evidence

- [discussion] The pathway loss rate represents the proportion of the original pathways (0% misidentification) significant at p <= 0.1 that are no longer significant at f % misidentification.: [evidence span withheld — anchor status: not_found]
- [discussion] Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean.: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation"
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: "Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k"
- [discussion] Another limitation is that in the majority of examples, a p-value threshold of P <= 0.1 was used without multiple testing correction to select SEPs.: [evidence span withheld — anchor status: not_found]
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
