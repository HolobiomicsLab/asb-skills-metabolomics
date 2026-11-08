---
name: p-value-multiple-testing-correction
description: Use when a pathway overrepresentation analysis produces a table of raw
  per-pathway p-values from Fisher's exact test (or Fisher's-method combined metabolite/gene
  p-values) and they must be corrected for multiple testing before ranking or cutoff
  filtering.
when_to_use_negative:
- Input is a single pre-specified hypothesis test with no multiplicity across pathways
  — correction would inflate the Holm/FDR adjusted p-values unnecessarily
- P-values come from a method other than Fisher's exact test on pathway contingency
  tables (e.g. permutation-based gene-set tests), where the RaMP FilterFishersResults
  pipeline's assumptions about the p-value source no longer hold
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: Provides the enrichment p-value computation (runFisherTest, runCombinedFisherTest)
    and the multiple-testing correction/filtering entry point FilterFishersResults
    over its KEGG/Reactome/WikiPathways/HMDB pathway annotations
  repo: https://github.com/Mathelab/RaMP-DB
- name: R
  role: Language in which the RaMP package functions for Fisher testing, BH/Holm adjustment,
    and result filtering are implemented
- name: MySQL
  role: Backend database accessed via connectToRaPM(dbname, username, conpass, host)
    with RMySQL/DBI that supplies the pathway-analyte contingency-table counts used
    by the Fisher tests
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:0cccc571f7db91aa8b4934ec07f8866682ca8ba2ef436b2e551f48ecc15675e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - MySQL
  - R
  - RMySQL
  - RaMP
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# p-value-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg FDR and Holm step-down corrections to raw pathway enrichment p-values computed by Fisher's exact test (metabolites, genes, or Fisher's-method combined p-values) in RaMP, then filter results at user-specified p-value cutoffs via FilterFishersResults.

## When to use

Use when a pathway overrepresentation analysis produces a table of raw per-pathway p-values from Fisher's exact test (or Fisher's-method combined metabolite/gene p-values) and they must be corrected for multiple testing before ranking or cutoff filtering.

## When NOT to use

- Input is a single pre-specified hypothesis test with no multiplicity across pathways — correction would inflate the Holm/FDR adjusted p-values unnecessarily
- P-values come from a method other than Fisher's exact test on pathway contingency tables (e.g. permutation-based gene-set tests), where the RaMP FilterFishersResults pipeline's assumptions about the p-value source no longer hold

## Inputs

- fishers_df: per-pathway enrichment results table with raw p-values from runFisherTest / runCombinedFisherTest (columns include pathway id, pathway name, metabolite p-value Pm, gene p-value Pg, combined p-value Pcomb)
- p_holmadj_cutoff: numeric Holm-adjusted p-value cutoff supplied by the user
- p_fdradj_cutoff: numeric Benjamini-Hochberg FDR-adjusted p-value cutoff supplied by the user

## Outputs

- Filtered enrichment results table with BH-FDR and Holm adjusted p-value columns retained alongside raw p-values
- fishers_results.csv written via write_FishersResults(fishResults, outputfile)

## How to apply

First compute raw p-values per pathway with runFisherTest (Fisher's exact test on each pathway's 2x2 contingency table of annotated vs. input analytes) and, when both metabolites and genes are supplied, combine the two independent p-values with runCombinedFisherTest using Fisher's combined probability test (Tcomb = -2 x [ln(Pm)+ln(Pg)] against a chi-square distribution with 2 degrees of freedom), with fallbacks Pcomb = Pg when Pm is missing and Pcomb = Pm when Pg is missing. Then pass the resulting fishers_df to FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff), which applies both the Benjamini-Hochberg FDR and Holm step-down corrections to the raw p-values and filters the table at the user-specified adjusted p-value cutoffs. Because the two corrections control different error rates (FDR vs. family-wise error), report both adjusted columns and select cutoffs appropriate to the intended claim. Finally write the filtered enrichment table to disk with write_FishersResults(fishResults, outputfile) as fishers_results.csv so the correction provenance is preserved alongside the raw values.

## Related tools

- **RaMP** (Provides the enrichment p-value computation (runFisherTest, runCombinedFisherTest) and the multiple-testing correction/filtering entry point FilterFishersResults over its KEGG/Reactome/WikiPathways/HMDB pathway annotations) — https://github.com/Mathelab/RaMP-DB
- **R** (Language in which the RaMP package functions for Fisher testing, BH/Holm adjustment, and result filtering are implemented)
- **MySQL** (Backend database accessed via connectToRaPM(dbname, username, conpass, host) with RMySQL/DBI that supplies the pathway-analyte contingency-table counts used by the Fisher tests)

## Examples

```
rampDB <- connectToRaPM(dbname, username, conpass, host); fishers_df <- runCombinedFisherTest(pathwaydf, total_metabolites, total_genes, min_analyte, conpass, dbname, username, host); filtered <- FilterFishersResults(fishers_df, p_holmadj_cutoff=0.05, p_fdradj_cutoff=0.05); write_FishersResults(filtered, outputfile="fishers_results.csv")
```

## Evaluation signals

- Adjusted p-values must be monotonically non-decreasing relative to the raw p-values within each correction (Holm and BH are monotone transformations)
- Holm-adjusted p-values should be >= BH-FDR-adjusted p-values for the same pathway (family-wise control is at least as conservative as FDR control)
- Every row in the filtered output must satisfy both p_holmadj_cutoff and p_fdradj_cutoff as supplied to FilterFishersResults
- The fallback logic must hold in the output: rows missing Pm should carry Pcomb = Pg, and rows missing Pg should carry Pcomb = Pm
- The final file on disk must be fishers_results.csv produced by write_FishersResults and must contain the adjusted p-value columns, not just the raw ones

## Limitations

- The combined p-value Pcomb requires both Pm and Pg; when one analyte type is missing the pipeline falls back to the single-type p-value, so combined significance cannot be claimed for pathways annotated with only one analyte type
- Result filtering is applied at user-specified p_holmadj_cutoff and p_fdradj_cutoff — no default significance threshold is imposed by the correction step itself
- The Mathelab/RaMP-DB repository is no longer maintained; users are directed to https://github.com/ncats/RaMP-DB/
- The min_analyte pathway-size filter applied during runFisherTest/runCombinedFisherTest determines which pathways even receive a p-value, so small pathways below that filter never enter the correction step

## Evidence

- [methods] For pathways annotated with both genes and metabolites, Fisher's method combines the independent metabolite and gene test p-values into a combined p-value (Pcomb): [evidence span withheld — anchor status: not_found]
- [methods] Tcomb formula against chi-square with 2 df: [evidence span withheld — anchor status: not_found]
- [methods] FilterFishersResults applies BH and Holm cutoffs: "FilterFishersResults(fishers_df, p_holmadj_cutoff, p_fdradj_cutoff)"
- [methods] Adjusting raw p-values with both BH FDR and Holm corrections: [evidence span withheld — anchor status: not_found]
- [readme] README confirms repo maintenance status: "# This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
