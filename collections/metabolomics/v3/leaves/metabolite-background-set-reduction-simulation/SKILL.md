---
name: metabolite-background-set-reduction-simulation
description: Use when evaluating the robustness of over-representation analysis results
  to the composition of the background/reference compound set, e.g. when the assay-specific
  background may be incomplete or when reporting how SEP proportions change as the
  background shrinks.
when_to_use_negative:
- The goal is to test metabolite misidentification, pathway database choice, or organism-specific
  pathway sets — those are separate ORA parameter perturbations, not background-set
  size reduction.
- 'You need a definitive ''ideal'' background set: the article states defining the
  ideal assay-specific background set remains an open problem, so this simulation
  quantifies sensitivity rather than prescribing a correct background.'
- The input is not a metabolomics abundance matrix with an accompanying assay-specific
  compound background list (e.g. it is already a list of significant pathways).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3172
tools:
- name: scipy stats fisher_exact
  role: Implements the right-tailed Fisher's exact test (hypergeometric) used to recompute
    pathway p-values after each background reduction step
- name: Python 3.8
  role: Language in which all ORA background-reduction simulations are implemented
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb, which contains the simulation code to
    adapt to one's own data
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative execution of the same simulations notebook without
    local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
- name: pip3
  role: Installs notebook dependencies from requirements.txt before running the simulations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:3afb338b18d89abd03d96f1e1da3033236469008fb513b24662f2b76487e7ae6
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolite-background-set-reduction-simulation

## Summary

In-silico simulation that iteratively shrinks an ORA background compound set (by random removal or non-DA-only removal) and recomputes right-tailed Fisher's exact tests to quantify how background size affects the proportion of significantly enriched pathways (SEPs). Grounded in Wieder et al. 2021's background-set sensitivity analysis of metabolomics ORA.

## When to use

Use when evaluating the robustness of over-representation analysis results to the composition of the background/reference compound set, e.g. when the assay-specific background may be incomplete or when reporting how SEP proportions change as the background shrinks.

## When NOT to use

- The goal is to test metabolite misidentification, pathway database choice, or organism-specific pathway sets — those are separate ORA parameter perturbations, not background-set size reduction.
- You need a definitive 'ideal' background set: the article states defining the ideal assay-specific background set remains an open problem, so this simulation quantifies sensitivity rather than prescribing a correct background.
- The input is not a metabolomics abundance matrix with an accompanying assay-specific compound background list (e.g. it is already a list of significant pathways).

## Inputs

- Processed metabolite abundance matrix with group labels per dataset
- Assay-specific background compound list per dataset
- Differentially abundant metabolite list (two-tailed t-tests, BH FDR q <= 0.05)
- Pathway compound sets from KEGG, Reactome (ChEBI-mapped), and BioCyc, restricted to pathways with at least 3 compounds

## Outputs

- Per-dataset background reduction tables/curves of retained SEP proportion (p <= 0.1) vs. remaining background fraction (e.g. background_reduction_sweep.csv)
- Mean SEP proportion and standard error of the mean over 100 randomised simulations, per dataset, removal mode, and reduction step

## How to apply

Load each dataset's processed abundance matrix with group labels, its assay-specific background compound list, the differentially abundant (DA) metabolite list (two-tailed Student's t-tests with Benjamini-Hochberg FDR at q <= 0.05), and pathway compound sets from KEGG/Reactome/BioCyc, keeping only pathways with at least 3 compounds. Run two removal sweeps: (1) random removal, iteratively discarding compounds at random down to 10% of the original background count while re-deriving the DA list at each step with BH-FDR-adjusted t-tests; (2) non-DA-only removal, discarding only non-DA compounds down to 10% while keeping the original DA list fixed. After each reduction step, recompute ORA with scipy.stats.fisher_exact (right-tailed, hypergeometric), calculating p-values when k >= 1, and count SEPs at p <= 0.1 without multiple-testing correction. Repeat each randomised simulation 100 times and record the mean retained SEP proportion with its standard error of the mean per dataset, removal mode, and reduction step.

## Related tools

- **scipy stats fisher_exact** (Implements the right-tailed Fisher's exact test (hypergeometric) used to recompute pathway p-values after each background reduction step)
- **Python 3.8** (Language in which all ORA background-reduction simulations are implemented) — https://github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb, which contains the simulation code to adapt to one's own data) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative execution of the same simulations notebook without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
- **pip3** (Installs notebook dependencies from requirements.txt before running the simulations)

## Examples

```
git clone https://github.com/cwieder/metabolomics-ORA.git && cd metabolomics-ORA/src && pip3 install -r requirements.txt && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- The retained SEP proportion should decline monotonically (or near-monotonically) as the background set shrinks toward 10% of the original compound count, in both removal modes.
- Datasets with larger initial background sets (e.g. Fuhrer et al., >240 metabolites remaining at 10%) should show the smallest decline in SEP proportion.
- With 100% of compounds retained, the recomputed ORA p-values must match the original analysis (baseline check: same DA list, same pathway sets, same right-tailed fisher_exact settings, k >= 1).
- Random-removal sweeps should yield lower SEP proportions than non-DA-only-removal at matched background sizes when the DA list is re-derived, because random removal also perturbs the DA list.
- Standard error of the mean across the 100 randomised repeats should shrink with larger retained background sizes; large SEMs at deep reductions indicate unstable simulations.

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so the simulations quantify sensitivity rather than correctness.
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs, which can inflate apparent SEP counts.
- Defining the ideal assay-specific background set remains an open problem; the skill demonstrates the effect of background reduction but cannot prescribe a correct background.
- Code tested only on Python 3.8 on MacOS (v11.2.3) with 16GB RAM standard hardware; results limited to five MS-based untargeted metabolomics datasets.

## Evidence

- [results] Both random removal and removal of non-DA compounds from the background set decreased the proportion of significantly enriched pathways: [evidence span withheld — anchor status: not_found]
- [discussion] ORA implementation via scipy fisher_exact: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Pathways restricted to at least 3 compounds: [evidence span withheld — anchor status: not_found]
- [discussion] Randomised simulations repeated 100 times: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [discussion] DA metabolites selected with BH FDR q <= 0.05: [evidence span withheld — anchor status: not_found]
- [readme] Notebook launch instructions from README: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
