---
name: monte-carlo-replicate-aggregation
description: Use when a simulation step contains randomisation — such as randomly
  selecting f% of compounds for isobaric or identical-formula replacement — and the
  resulting metric (e.g. pathway loss/gain rate) must be reported with a stable central
  estimate and uncertainty.
when_to_use_negative:
- The analysis is fully deterministic (no randomised selection or replacement), so
  a single run suffices and replicate averaging would add no information
- The target quantity needs a full sampling distribution, confidence intervals, or
  hypothesis tests over replicates rather than a mean with standard error of the mean
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: Python 3.8
  role: Implementation language for the 100-replicate simulation loop and aggregation
    of means and standard errors
  repo: https://github.com/cwieder/metabolomics-ORA
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test re-run on every perturbed compound list within
    each replicate
- name: Jupyter Notebook
  role: Hosts reproducible_simulations.ipynb, the notebook in which the repeated simulations
    are executed and results reported
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative to run the same 100-replicate simulations without
    local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:c9044f07adff6a426b1534b7403e5e5bd56024137a92b184a24d626a88fe8f33
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# monte-carlo-replicate-aggregation

## Summary

Repeats a randomised in-silico perturbation (e.g. metabolite misidentification at a set misidentification rate) 100 times and reports the mean and standard error of the mean of each outcome metric, so that stochastic pathway-level effects are quantified rather than relying on a single random realisation.

## When to use

Use when a simulation step contains randomisation — such as randomly selecting f% of compounds for isobaric or identical-formula replacement — and the resulting metric (e.g. pathway loss/gain rate) must be reported with a stable central estimate and uncertainty.

## When NOT to use

- The analysis is fully deterministic (no randomised selection or replacement), so a single run suffices and replicate averaging would add no information
- The target quantity needs a full sampling distribution, confidence intervals, or hypothesis tests over replicates rather than a mean with standard error of the mean

## Inputs

- Perturbed compound lists from each of the 100 randomised misidentification replicates (isobaric ±20 ppm or identical-chemical-formula replacements)
- Baseline (0% misidentification) ORA results: pathways significant at p <= 0.1 per dataset
- Per-replicate ORA p-values from the right-tailed Fisher's exact test (scipy stats fisher_exact)

## Outputs

- Mean pathway loss rate and pathway gain rate per dataset per replacement mode, aggregated over 100 random samplings
- Standard error of the mean for each aggregated rate
- Per-dataset summary table of misidentification effects (e.g. at 4% misidentification, as in Fig 5)

## How to apply

Wrap the randomised perturbation in a loop of 100 independent replicates: each replicate re-draws the random selection of compounds (e.g. 4% of compounds with an isobaric counterpart within ±20 ppm, or a replacement of identical chemical formula present in at least one KEGG pathway and absent from the background list), re-runs the downstream analysis (right-tailed Fisher's exact test ORA via scipy stats fisher_exact), and recomputes the per-replicate outcome (e.g. pathway loss rate and gain rate at p <= 0.1 using Eqs 2-3). After all replicates, aggregate by computing the mean and the standard error of the mean per dataset and per replacement mode. The rationale is that a single random sampling of misidentified compounds could by chance shift ORA results; averaging over 100 samplings yields a reproducible estimate of the effect size. Implement this in the reproducible_simulations.ipynb Jupyter notebook (or its Google Colab version) from github.com/cwieder/metabolomics-ORA, tested with Python 3.8.

## Related tools

- **Python 3.8** (Implementation language for the 100-replicate simulation loop and aggregation of means and standard errors) — https://github.com/cwieder/metabolomics-ORA
- **scipy stats fisher_exact** (Right-tailed Fisher's exact test re-run on every perturbed compound list within each replicate)
- **Jupyter Notebook** (Hosts reproducible_simulations.ipynb, the notebook in which the repeated simulations are executed and results reported) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative to run the same 100-replicate simulations without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk

## Examples

```
jupyter-notebook reproducible_simulations.ipynb  # run src/reproducible_simulations.ipynb cells to execute the 100-replicate misidentification simulations
```

## Evaluation signals

- Number of replicates per randomised condition equals 100, matching the paper's protocol
- Aggregated results are reported as mean of 100 random samplings alongside the standard error of the mean, per dataset and per replacement mode (±20 ppm and identical formula)
- At 4% misidentification, all five datasets show nonzero pathway loss and gain rates under both replacement modes, reproducing the Fig 5 finding
- Loss and gain rates are computed against the 0% misidentification baseline of pathways significant at p <= 0.1
- Re-running the notebook reproduces the same aggregated means and SEMs given the same random seeds/setup

## Limitations

- Only a mean and standard error of the mean are reported; the study notes that in most examples a p-value threshold of P <= 0.1 was used without multiple testing correction to select significantly enriched pathways
- No ground-truth dataset exists where the identities of enriched pathways are known, so gain of 'false-positive' pathways cannot be directly validated against truth
- No benchmark level of metabolite misidentification typical of metabolomics studies exists to anchor the choice of misidentification rates
- Study limited to five MS-based untargeted metabolomics datasets; results may not generalise to other assay types

## Evidence

- [discussion] Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean.: "Simulations with an element of randomisation were repeated 100 times, and results are reported as the mean of 100 random samplings of the simulation, alongside the standard error of the mean."
- [discussion] ORA implementation via custom Python script: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Pathway loss rate definition: [evidence span withheld — anchor status: not_found]
- [readme] Notebook usage: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**. Users may adapt the code in the notebook to perform the simulations on their"
- [abstract] 4% misidentification finding: "Simulated metabolite misidentification rates as low as 4% resulted in both gain of false-positive pathways and loss of truly significant pathways across all datasets."
