---
name: fishers-exact-hypergeometric-enrichment-testing
description: Use when a list of differentially abundant metabolites (metabolites of
  interest) must be tested against a pathway set and background set to obtain per-pathway
  enrichment p-values and identify significantly enriched pathways.
when_to_use_negative:
- The analysis goal is GSEA-style ranking or network/topology-based pathway analysis
  rather than count-based over-representation of a selected metabolite list.
- No pathway set or background compound list has been assembled — ORA requires all
  three inputs (pathways, metabolites of interest, background set) before Fisher's
  exact testing can be run.
- The task is database-level comparison of pathway set properties (e.g. overlap coefficients
  between KEGG human and reference sets) rather than per-pathway enrichment testing
  on a dataset.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3172
tools:
- name: scipy stats fisher_exact
  role: Computes the right-tailed (hypergeometric) p-value for each pathway's 2x2
    contingency table in the custom ORA Python script.
- name: Python 3.8
  role: Runtime environment in which the ORA custom script and reproducible_simulations.ipynb
    are executed.
- name: Jupyter Notebook
  role: reproducible_simulations.ipynb wraps the ORA simulation code; users adapt
    its cells to run the same Fisher-exact ORA on their own data.
- name: metabolomics-ORA
  role: Repository containing src/reproducible_simulations.ipynb and requirements.txt
    that implements the ORA simulations.
  repo: https://github.com/cwieder/metabolomics-ORA.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:e884f42962791038f5676a568fbf8df244fa2b02c5021a26b43e8e79018ee88a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA.git
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8
  - metabolomics-ORA
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fishers-exact-hypergeometric-enrichment-testing

## Summary

Performs over-representation analysis (ORA) by scoring each pathway with a right-tailed Fisher's exact test based on the hypergeometric distribution, using scipy.stats.fisher_exact in Python, as implemented in the metabolomics-ORA simulations of Wieder et al. 2021.

## When to use

Use when a list of differentially abundant metabolites (metabolites of interest) must be tested against a pathway set and background set to obtain per-pathway enrichment p-values and identify significantly enriched pathways.

## When NOT to use

- The analysis goal is GSEA-style ranking or network/topology-based pathway analysis rather than count-based over-representation of a selected metabolite list.
- No pathway set or background compound list has been assembled — ORA requires all three inputs (pathways, metabolites of interest, background set) before Fisher's exact testing can be run.
- The task is database-level comparison of pathway set properties (e.g. overlap coefficients between KEGG human and reference sets) rather than per-pathway enrichment testing on a dataset.

## Inputs

- Collection of pathways (or custom metabolite sets), e.g. KEGG compound-only pathway set, restricted to pathways with >= 3 compounds
- List of metabolites of interest (differentially abundant metabolite identifiers, e.g. KEGG/ChEBI/BioCyc IDs)
- Background (assay-specific) compound set

## Outputs

- Per-pathway enrichment p-values (right-tailed Fisher's exact / hypergeometric)
- List of significantly enriched pathways (SEPs) at a chosen threshold, e.g. p <= 0.1
- sep_counts_table.csv-style summary of significant pathway counts per dataset/pathway set

## How to apply

Assemble the three ORA inputs — a collection of pathways (e.g., KEGG, Reactome ChEBI-mapped, or BioCyc compound sets), the list of metabolites of interest, and the background (assay-specific) compound set — then compute, for each pathway, a 2x2 contingency table of counts (metabolites of interest in/not-in the pathway versus background in/not-in the pathway). Run scipy.stats.fisher_exact on that table with a right-tailed (alternative='greater') setup so the hypergeometric one-sided p-value reflects over-representation. Restrict input to pathways containing at least 3 compounds, and calculate a p-value only when the parameter k (number of differentially abundant metabolites in the pathway) is >= 1; otherwise leave the pathway un-scored. Compare the resulting p-values against a significance threshold (the study used p <= 0.1 without multiple testing correction for most examples) to assemble the final list of significantly enriched pathways, and optionally apply BH FDR or Bonferroni correction as a sensitivity variant.

## Related tools

- **scipy stats fisher_exact** (Computes the right-tailed (hypergeometric) p-value for each pathway's 2x2 contingency table in the custom ORA Python script.)
- **Python 3.8** (Runtime environment in which the ORA custom script and reproducible_simulations.ipynb are executed.)
- **Jupyter Notebook** (reproducible_simulations.ipynb wraps the ORA simulation code; users adapt its cells to run the same Fisher-exact ORA on their own data.)
- **metabolomics-ORA** (Repository containing src/reproducible_simulations.ipynb and requirements.txt that implements the ORA simulations.) — https://github.com/cwieder/metabolomics-ORA.git

## Examples

```
from scipy.stats import fisher_exact; p = fisher_exact([[k, m-k], [n-m, bg-n-m+k]], alternative='greater')
```

## Evaluation signals

- Every pathway used as ORA input contains at least 3 compounds, and p-values are only emitted when k (differentially abundant metabolites in the pathway) >= 1.
- The contingency table fed to scipy.stats.fisher_exact is right-tailed so that over-representation (not under-representation) is being tested.
- Pathway p-values reproduce the study's direction of results, e.g. sep_counts_table.csv shows many more significantly enriched pathways with the KEGG reference set than the organism-specific KEGG human set.
- The final SEP list is thresholded consistently (p <= 0.1 without multiple testing correction in the reference workflow) and matches the number of pathways tested.
- Randomised simulations (e.g. misidentification swaps) are repeated 100 times with mean and standard error of the mean reported rather than a single run.

## Limitations

- In the majority of examples, a p-value threshold of p <= 0.1 was used without multiple testing correction, inflating the SEP count; applying BH FDR or Bonferroni is a sensitivity variant, not the default in this workflow.
- Lack of a ground-truth dataset means the true enriched pathway identities are unknown, so ORA accuracy cannot be directly validated.
- ORA results depend heavily on upstream choices (background set, differential metabolite selection threshold, pathway database), so p-values from this test alone do not guarantee robust biological conclusions.

## Evidence

- [intro] Right-tailed Fisher's exact test p-values: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [discussion] ORA implemented with scipy: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Minimum pathway size and k >= 1 rules: [evidence span withheld — anchor status: not_found]
- [intro] Three ORA inputs required: "To perform ORA, three essential inputs are required: a collection of pathways (or custom metabolite sets), a list of metabolites of interest, and a background or reference set of compounds."
- [readme] Notebook entry point: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**."
