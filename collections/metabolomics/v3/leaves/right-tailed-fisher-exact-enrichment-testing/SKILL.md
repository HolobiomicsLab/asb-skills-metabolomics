---
name: right-tailed-fisher-exact-enrichment-testing
description: Use when you have a list of metabolites of interest, a collection of
  pathway compound sets, and a background set, and need per-pathway enrichment p-values
  to identify significantly enriched pathways (SEPs).
when_to_use_negative:
- When the goal is relative position of whole metabolite lists along a pathway (e.g.
  GSEA-style rank-based pathway analysis) rather than over-representation of a thresholded
  metabolite-of-interest list
- When no valid background/reference compound set can be defined for the assay — the
  article notes that defining the ideal assay-specific background remains an open
  problem
- When pathway sets contain fewer than 3 compounds, which this pipeline explicitly
  excludes as ORA input
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scipy stats fisher_exact
  role: Computes the right-tailed (hypergeometric) pathway p-values for ORA in a custom
    Python script
  repo: github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Runs the reproducible_simulations.ipynb notebook that executes the ORA simulations
    on the five datasets
  repo: github.com/cwieder/metabolomics-ORA
- name: MetaboAnalyst 4.0
  role: Converts compound names in the input metabolomics datasets to KEGG identifiers
    prior to ORA
- name: bioservices (Python package) v 1.7.1
  role: Maps KEGG compounds to ChEBI identifiers to support Reactome pathway ORA
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:d833a634bb691c9f0f59ffb7fca3d1ba75f24ea8f40ad6cc2321e21690c32e17
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - MetaboAnalyst
  - MetaboAnalyst 4.0
  - Python
  - bioservices (Python package) v 1.7.1
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# Right-tailed Fisher's exact enrichment testing for metabolomics ORA

## Summary

Implements over-representation analysis (ORA) for metabolomics by scoring each pathway with a right-tailed Fisher's exact test (hypergeometric distribution) via scipy.stats.fisher_exact, quantifying whether differentially abundant metabolites are over-represented in pathway compound sets.

## When to use

Use when you have a list of metabolites of interest, a collection of pathway compound sets, and a background set, and need per-pathway enrichment p-values to identify significantly enriched pathways (SEPs).

## When NOT to use

- When the goal is relative position of whole metabolite lists along a pathway (e.g. GSEA-style rank-based pathway analysis) rather than over-representation of a thresholded metabolite-of-interest list
- When no valid background/reference compound set can be defined for the assay — the article notes that defining the ideal assay-specific background remains an open problem
- When pathway sets contain fewer than 3 compounds, which this pipeline explicitly excludes as ORA input

## Inputs

- Pathway compound sets from KEGG (release 96), Reactome (via ChEBI identifiers), or BioCyc/HumanCyc, filtered to pathways with >= 3 compounds
- List of differentially abundant metabolites of interest (e.g. BH FDR q <= 0.05) mapped to database identifiers (KEGG, ChEBI/Reactome, BioCyc)
- Background/reference compound set (assay-specific, or the nonspecific KEGG human 3373-compound set for comparison)

## Outputs

- Per-pathway right-tailed Fisher's exact p-values for each dataset/background/database combination
- Table of significantly enriched pathways (SEPs) at p <= 0.1 uncorrected and q <= 0.1 after FDR correction
- Comparison files such as pvalue_comparison.csv and sep_count_comparison.csv for contrasting background settings

## How to apply

Assemble three ORA inputs: pathway compound sets (e.g. KEGG release 96, Reactome, or BioCyc/HumanCyc), a list of metabolites of interest (e.g. differentially abundant metabolites selected at BH FDR q <= 0.05 from two-tailed t-tests), and a background set (assay-specific rather than nonspecific organism-wide, since a nonspecific background inflates false positives). Restrict pathway sets to those containing at least 3 compounds, and compute a pathway p-value only when k (the number of metabolites of interest mapping to the pathway) is >= 1, using scipy.stats.fisher_exact with a right-tailed (hypergeometric) alternative. Apply multiple testing correction (e.g. BH FDR) and call SEPs at a stated threshold (in this study p <= 0.1 uncorrected or q <= 0.1 corrected). Tabulate per-pathway p-values and SEP counts per dataset and background/database setting so parameter effects (background set, database choice, misidentification) can be compared.

## Related tools

- **scipy stats fisher_exact** (Computes the right-tailed (hypergeometric) pathway p-values for ORA in a custom Python script) — github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Runs the reproducible_simulations.ipynb notebook that executes the ORA simulations on the five datasets) — github.com/cwieder/metabolomics-ORA
- **MetaboAnalyst 4.0** (Converts compound names in the input metabolomics datasets to KEGG identifiers prior to ORA)
- **bioservices (Python package) v 1.7.1** (Maps KEGG compounds to ChEBI identifiers to support Reactome pathway ORA)

## Examples

```
python -c "from scipy.stats import fisher_exact; p = fisher_exact([[k, K-k], [m-k, N-K-m+k]], alternative='greater')"
```

## Evaluation signals

- Verify that no p-value is computed for pathways where k = 0 (no differentially abundant metabolites map to the pathway); the rule is to calculate a p-value only when k >= 1
- Check that all pathway sets retained for ORA contain at least 3 compounds
- Confirm the test is right-tailed (alternative='greater'), matching the hypergeometric over-representation formulation
- Reproduce the expected pattern: far fewer SEPs under the assay-specific background than the nonspecific 3373-compound KEGG human background, with two datasets showing zero SEPs after q <= 0.1 correction
- Verify that SEP counts are tabulated per dataset and per database (KEGG/Reactome/HumanCyc) and written to ora_concordance_table.csv / sep_count_comparison.csv with plausible counts

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so SEP correctness cannot be validated absolutely
- In most examples a p <= 0.1 threshold was used without multiple testing correction to select SEPs, which can inflate SEP counts
- The use of a non-assay-specific background set produces large numbers of false-positive pathways; the ideal assay-specific background set definition remains unresolved
- Results depend strongly on parameters (background set, differential metabolite selection, pathway database), so conclusions are not transferable across settings

## Evidence

- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values.: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k: [evidence span withheld — anchor status: not_found]
- [intro] P-values for each pathway are calculated using a right-tailed Fisher's exact test based on the hypergeometric distribution.: "P-values for each pathway are calculated using a right-tailed Fisher’s exact test based on the hypergeometric distribution."
- [readme] The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**.: "The Python code to generate the results is contained within the Jupyter notebook **src/reproducible_simulations.ipynb**. Users may adapt the code in the notebook to perform the simulations on their"
