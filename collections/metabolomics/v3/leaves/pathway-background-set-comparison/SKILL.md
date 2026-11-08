---
name: pathway-background-set-comparison
description: Use when performing over-representation analysis on metabolomics data
  and you must decide whether the background/reference compound set should be assay-specific
  or a database-wide organism set, or must quantify that choice's effect on p-values
  and SEP counts.
when_to_use_negative:
- The goal is to compare pathway databases (KEGG vs Reactome vs BioCyc) or other ORA
  parameters rather than the effect of the background set
- Only one background set is available for the dataset, so no nonspecific-vs-assay-specific
  pairing can be constructed
- Ground-truth identities of enriched pathways are required for validating ORA output
  — the study notes no such ground-truth dataset exists
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0204
tools:
- name: scipy stats fisher_exact
  role: Computes right-tailed (hypergeometric) Fisher's exact test p-values per pathway
    under each background setting
- name: MetaboAnalyst 4.0
  role: Converts compound names in each dataset to KEGG identifiers before ORA background
    mapping
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb, which executes the paired-background
    ORA and writes the comparison CSVs
  repo: https://github.com/cwieder/metabolomics-ORA
- name: pip3
  role: Installs notebook dependencies via requirements.txt before running the comparison
  repo: https://github.com/cwieder/metabolomics-ORA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:9368c0aa109a57f9d5d0d3336d8f4036304332340d68fc9f521ff80321395a79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Jupyter Notebook
  - MetaboAnalyst
  - MetaboAnalyst 4.0
  - Python
  - pip3
  - scipy stats fisher_exact
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pathway-background-set-comparison

## Summary

Quantifies how swapping a nonspecific (e.g., KEGG organism-wide, 3373 compounds) background for an assay-specific background changes ORA pathway p-values and the count of significantly enriched pathways (SEPs), revealing false positives introduced by an over-large reference set.

## When to use

Use when performing over-representation analysis on metabolomics data and you must decide whether the background/reference compound set should be assay-specific or a database-wide organism set, or must quantify that choice's effect on p-values and SEP counts.

## When NOT to use

- The goal is to compare pathway databases (KEGG vs Reactome vs BioCyc) or other ORA parameters rather than the effect of the background set
- Only one background set is available for the dataset, so no nonspecific-vs-assay-specific pairing can be constructed
- Ground-truth identities of enriched pathways are required for validating ORA output — the study notes no such ground-truth dataset exists

## Inputs

- List of differentially abundant metabolites per dataset as KEGG identifiers (selected at BH FDR q <= 0.05)
- Assay-specific background compound list (all compounds measured by the platform)
- Nonspecific KEGG organism-wide background set (3373 human compounds)
- KEGG release 96 human pathway sets filtered to pathways containing at least 3 compounds

## Outputs

- pvalue_comparison.csv — paired pathway p-values across the two background settings (plotted as Figure 2A)
- sep_count_comparison.csv — counts of significantly enriched pathways at p <= 0.1 (uncorrected) and q <= 0.1 (FDR-corrected) per background setting (plotted as Figure 2B)

## How to apply

Convert metabolite identifiers to KEGG IDs (e.g., via the MetaboAnalyst 4.0 ID conversion tool) and assemble KEGG release 96 human pathway sets, retaining only pathways with at least 3 compounds. Take each dataset's differentially abundant metabolite list (two-tailed t-tests with Benjamini-Hochberg FDR, selecting at q <= 0.05) as the metabolites of interest. Build the two background settings: the nonspecific KEGG human background (3373 compounds) and the dataset's own assay-specific background. In reproducible_simulations.ipynb, run right-tailed Fisher's exact test (hypergeometric) ORA per dataset under both backgrounds via scipy.stats.fisher_exact, computing a pathway p-value only when k >= 1 metabolite of interest maps to the pathway. Tabulate paired p-values (Figure 2A) and SEP counts at p <= 0.1 uncorrected and q <= 0.1 after FDR correction (Figure 2B), saving pvalue_comparison.csv and sep_count_comparison.csv.

## Related tools

- **scipy stats fisher_exact** (Computes right-tailed (hypergeometric) Fisher's exact test p-values per pathway under each background setting)
- **MetaboAnalyst 4.0** (Converts compound names in each dataset to KEGG identifiers before ORA background mapping)
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb, which executes the paired-background ORA and writes the comparison CSVs) — https://github.com/cwieder/metabolomics-ORA
- **pip3** (Installs notebook dependencies via requirements.txt before running the comparison) — https://github.com/cwieder/metabolomics-ORA

## Examples

```
cd metabolomics-ORA/src && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- The SEP-count table shows far fewer significantly enriched pathways under the assay-specific background than under the nonspecific KEGG background, at both p <= 0.1 and q <= 0.1 thresholds
- Two datasets show zero SEPs after FDR correction at q <= 0.1 under the assay-specific background
- A pathway p-value is produced only when k >= 1 metabolite of interest maps to the pathway (no p-value computed for k = 0)
- p-values match a right-tailed Fisher's exact / hypergeometric calculation with the same background and pathway-set sizes on both sides of the comparison
- pvalue_comparison.csv contains paired p-values per pathway across both backgrounds, and sep_count_comparison.csv reports counts at both the uncorrected and FDR-corrected thresholds

## Limitations

- No ground-truth dataset exists where the identities of enriched pathways are known, so gains/losses of SEPs cannot be definitively labelled true or false positives
- In most examples a p <= 0.1 threshold without multiple testing correction was used to select SEPs, inflating apparent enrichment
- Defining the ideal assay-specific background set for a particular dataset remains an open problem

## Evidence

- [abstract] The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways.: "The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways."
- [discussion] ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values.: "ORA was implemented using a custom Python script that utilised the scipy stats fisher_exact function (right-tailed) to calculate pathway p-values."
- [discussion] Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k (number of differentially abundant metabolites in the ith pathway) was >= 1.: "Only pathways containing at least 3 compounds were used as input for ORA. p-values were calculated if the parameter k"
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"
- [abstract] changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results: "changes in parameters such as the background set, differential metabolite selection methods, and pathway database used can result in profoundly different ORA results"
