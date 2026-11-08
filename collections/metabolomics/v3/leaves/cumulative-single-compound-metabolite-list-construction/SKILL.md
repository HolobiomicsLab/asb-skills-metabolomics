---
name: cumulative-single-compound-metabolite-list-construction
description: Use when the goal is to probe how sensitive ORA output is to the composition
  and size of the differential metabolite list, e.g. to find the list size that maximises
  detected enriched pathways or to expose local extrema caused by single compounds.
when_to_use_negative:
- When the goal is to quantify robustness to metabolite misidentification or background-set
  changes rather than DA-list composition
- When the dataset is ground-truth benchmarked (article notes no ground-truth dataset
  where enriched pathway identities are known was available)
- When multiple-testing correction for SEP selection is required, since the article
  used p <= 0.1 without correction as a stated limitation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0154
tools:
- name: Python 3.8
  role: Runtime for computing t-tests and running the ORA simulations
- name: scipy stats fisher_exact
  role: Right-tailed Fisher's exact test used to calculate pathway p-values for each
    cumulative DA list
- name: Jupyter Notebook
  role: Hosts the reproducible_simulations.ipynb code that builds cumulative lists,
    runs ORA, and generates sensitivity plots
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same simulations notebook without
    local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:4292096d179276b0c7c36531de0b9af2823d498b8cd0f27010cfb0de4ed09169
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cumulative-single-compound-metabolite-list-construction

## Summary

Builds candidate lists of differentially abundant (DA) metabolites by adding compounds one at a time in ascending two-tailed t-test p-value order, then runs right-tailed Fisher's exact ORA on each to quantify how DA-list composition drives the number of significantly enriched pathways (SEPs).

## When to use

Use when the goal is to probe how sensitive ORA output is to the composition and size of the differential metabolite list, e.g. to find the list size that maximises detected enriched pathways or to expose local extrema caused by single compounds.

## When NOT to use

- When the goal is to quantify robustness to metabolite misidentification or background-set changes rather than DA-list composition
- When the dataset is ground-truth benchmarked (article notes no ground-truth dataset where enriched pathway identities are known was available)
- When multiple-testing correction for SEP selection is required, since the article used p <= 0.1 without correction as a stated limitation

## Inputs

- Five processed metabolomics datasets (post-imputation, log2-transformed, auto-scaled abundance matrices)
- Two-tailed Student's t-test p-values per metabolite between condition groups
- Pathway sets (KEGG, Reactome, BioCyc) restricted to pathways with at least 3 compounds

## Outputs

- sep_counts_table.csv tabulating SEP counts by multiple-testing correction method and alpha threshold
- Per-dataset sensitivity curves of SEP count versus cumulative DA list composition
- List of significantly enriched pathways at p <= 0.1 for each cumulative DA list

## How to apply

Load the processed metabolomics datasets (missing values imputed as half of each metabolite's minimum, log2-transformed, auto-scaled) and compute two-tailed Student's t-tests between condition groups for every metabolite in Python 3.8. Construct cumulative candidate DA lists by adding one compound at a time in ascending t-test p-value order, so each successive list grows by exactly one metabolite. For each candidate list, run right-tailed Fisher's exact test (hypergeometric) ORA via scipy.stats.fisher_exact, restricting pathways to those containing at least 3 compounds and computing a pathway p-value only when k >= 1 DA metabolite maps to the pathway. Count SEPs at p <= 0.1 without multiple testing correction for each list, tabulate counts by correction method and alpha threshold (0.005, 0.05, 0.1, with the default q <= 0.05 BH FDR selection) into sep_counts_table.csv, and plot per-dataset sensitivity curves of SEP count against DA list construction. The rationale is that the article shows the SEP count rises to a global maximum then falls to zero as the DA list approaches the full background set, so cumulative single-compound addition directly traces this dependence.

## Related tools

- **Python 3.8** (Runtime for computing t-tests and running the ORA simulations)
- **scipy stats fisher_exact** (Right-tailed Fisher's exact test used to calculate pathway p-values for each cumulative DA list)
- **Jupyter Notebook** (Hosts the reproducible_simulations.ipynb code that builds cumulative lists, runs ORA, and generates sensitivity plots) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same simulations notebook without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- Each per-dataset sensitivity curve shows a rise-then-fall SEP-count trend as the DA list grows, reaching a global maximum before falling to zero when the DA list equals the background set
- Single compounds cause local extrema (local maxima/minima) on each curve, identifiable by inspecting which metabolite was added at that list size
- SEP counts follow the Bonferroni < BH FDR ordering with the BH FDR maximum at alpha <= 0.05 across the correction and threshold grid
- Pathway p-values are only computed when k >= 1 DA metabolite maps to the pathway, and only pathways with >= 3 compounds enter the ORA input
- Results are reproducible via reproducible_simulations.ipynb (Python 3.8, 16GB RAM standard hardware, run in under 10 minutes after pip3 install -r requirements.txt)

## Limitations

- No ground-truth dataset available where the identities of enriched pathways are known
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs
- Study limited to five MS-based untargeted metabolomics datasets; two lacked a chromatographic separation step, implying higher uncertainty in metabolite identifications

## Evidence

- [results] cumulative single-compound list construction drives SEP count: [evidence span withheld — anchor status: not_found]
- [discussion] right-tailed Fisher exact ORA implementation: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] pathway size and k>=1 filter: [evidence span withheld — anchor status: not_found]
- [discussion] default DA metabolite selection threshold: [evidence span withheld — anchor status: not_found]
- [discussion] SEP selection threshold used in simulations: [evidence span withheld — anchor status: not_found]
- [discussion] post-processing of abundance matrices: "Missing abundance values were imputed using the minimum value of each metabolite divided by 2. All abundance values in the matrix were then log2 transformed and features (metabolites) were auto-scaled"
