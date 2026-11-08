---
name: missing-value-filtering-metabolomics-intensities
description: Use when a metabolomics abundance table (e.g., a Metabolomics Workbench
  study retrieved with getmwstudies) contains missing intensity values that would
  break per-metabolite t-test/ANOVA computation before enrichment analysis.
when_to_use_negative:
- The intensity matrix is already complete (no missing values) and the metabolite
  list will not be altered — filtering adds nothing and remove_NAs/50percent would
  be no-ops
- The goal is pathway or metabolite-class enrichment scoring (metclassenrichment,
  path_enrichmentscore) on already-identified significant metabolites rather than
  producing the significant-metabolite table itself
- The input is not metabolite abundance data with sample-level grouping metadata (e.g.,
  no factor_col defining two comparison groups), since fold change and group-wise
  testing cannot be defined
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetENP
  role: Provides the significant_met function whose three NA-handling variants (half_of_min,
    remove_NAs, 50percent) implement this filtering before t-test computation
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Language/runtime in which MetENP's significant_met and the NA-filtering variants
    are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:7909c597b60b8aa8cdfc2568155660a67141b100898d5925be73b81a830f41a9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Missing-value filtering for metabolomics intensities

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Prepares a metabolite abundance table for per-metabolite significance testing by handling missing intensity values via MetENP's three NA-filtering variants (half_of_min, remove_NAs, 50percent) inside the significant_met workflow, so that fold change, log2 fold change, p-value, and adjusted p-value are computed on a consistently filtered matrix.

## When to use

Use when a metabolomics abundance table (e.g., a Metabolomics Workbench study retrieved with getmwstudies) contains missing intensity values that would break per-metabolite t-test/ANOVA computation before enrichment analysis.

## When NOT to use

- The intensity matrix is already complete (no missing values) and the metabolite list will not be altered — filtering adds nothing and remove_NAs/50percent would be no-ops
- The goal is pathway or metabolite-class enrichment scoring (metclassenrichment, path_enrichmentscore) on already-identified significant metabolites rather than producing the significant-metabolite table itself
- The input is not metabolite abundance data with sample-level grouping metadata (e.g., no factor_col defining two comparison groups), since fold change and group-wise testing cannot be defined

## Inputs

- Metabolite abundance/intensity table from a Metabolomics Workbench study (via getmwstudies), with a metabolite identifier column (met_col) and sample columns
- Sample metadata with a factor column (factor_col) defining comparison groups factor1 and factor2 and sample identifiers (sample_col)
- Choice of NA-handling method: half_of_min, remove_NAs, or 50percent
- Choice of p-value adjustment method: holm, hochberg, hommel, or BH

## Outputs

- Per-metabolite results table (significant_metabolites.csv) with metabolite identifier, fold change, log2 fold change, p-value, and p_adjust-adjusted p-value
- Filtered intensity matrix in which metabolites excluded by the chosen NA rule no longer appear

## How to apply

Retrieve the study's metabolite abundance table and metadata from Metabolomics Workbench with getmwstudies(studyid, typedata), then invoke MetENP's significant_met(metabolomics_data, met_col, analysis_type, metadata, normalization, factor1, factor2, factor_col, sample_col, p_adjust) with one of three NA-handling choices. half_of_min imputes each missing intensity as half the minimum value observed for that metabolite, preserving all rows; remove_NAs drops any metabolite carrying at least one NA, keeping only complete cases; 50percent drops metabolites with more than 50% missing intensities. The choice is a decision point: imputation retains low-abundance metabolites, while the 50percent filter balances retention against statistical reliability. After filtering, significant_met runs the chosen test (e.g., Student's t-test) between factor1 and factor2 groups defined by factor_col, computes fold change and log2 fold change, and attaches raw and adjusted p-values using the selected adjustment method (holm, hochberg, hommel, or BH). Source the function via source('path/significant_met.R') if loading the uninstalled scripts from the MetENP GitHub repository.

## Related tools

- **MetENP** (Provides the significant_met function whose three NA-handling variants (half_of_min, remove_NAs, 50percent) implement this filtering before t-test computation) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Language/runtime in which MetENP's significant_met and the NA-filtering variants are executed)

## Examples

```
sig <- significant_met(metabolomics_data, met_col, analysis_type="t-test", metadata, normalization, factor1, factor2, factor_col, sample_col, p_adjust="BH")
```

## Evaluation signals

- significant_metabolites.csv exists, is a valid CSV, and contains the required fields: metabolite identifier, fold change, log2 fold change, p-value, and adjusted p-value
- The adjusted p-value column corresponds to the selected p_adjust method (holm, hochberg, hommel, or BH)
- Under remove_NAs, the retained row count equals the number of metabolites with zero missing intensities; under 50percent, no retained metabolite has more than 50% missing intensities
- Under half_of_min, no NA remains in the intensity matrix passed to the statistical test
- Fold change and log2 fold change for each metabolite are consistent with the factor1/factor2 group means defined by factor_col

## Limitations

- The three NA-handling variants are mutually exclusive choices with different biases: half_of_min imputation can distort low-abundance metabolites, remove_NAs is aggressive for sparse tables, and 50percent discards moderately sparse metabolites
- MetENP/MetENPWeb has no stated software version number, changelog, or license in the article text, so reproducibility of the filtering behavior depends on the GitHub checkout used
- No statistical validation metrics or quantitative benchmark results are reported for the three case studies, so filtering choices are validated only by workflow consistency

## Evidence

- [other] Missing values are handled via three filtering methods: half_of_min (replaces NAs with half the minimum value), remove_NAs (removes metabolites with NAs): [evidence span withheld — anchor status: not_found]
