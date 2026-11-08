---
name: peak-area-to-concentration-conversion
description: Use when the input is a lipidomic peak-area/peak-height matrix whose
  rows include internal standard species and a companion standards table (Standard/Class/Conc.)
  exists, and the goal is a per-species concentration matrix.
when_to_use_negative:
- Input data are already concentration values rather than raw peak areas/heights —
  re-applying conversion factors would double-normalize
- The dataset lacks internal standard rows or a standards table (no Standard/Class/Conc.
  mapping), so class-specific conversion factors cannot be constructed
- The goal is downstream statistical analysis (mono-/multivariate tests, biomarker
  identification) on already-quantified data — use LipidOne 2.0's data-mining functions
  instead of the quantification utility
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: LipidOne 2.0
  role: Web platform hosting the Lipid Data Quantification utility (Utility section)
    that performs the peak-area-to-concentration conversion and post-quantification
    matrix cleanup
  repo: https://lipidone.eu
- name: R
  role: Back-end scripting language and reproduction environment for the quantification
    workflow (loading data frames, building class-specific conversion factors, filtering
    standard rows, consolidating duplicates)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:c9c00b009e26b7748d76e493d51bfbd4fdd93671c09a4441752a268eb7a3b525
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peak-area-to-concentration-conversion

## Summary

Converts MS-derived peak area (or peak height) values into concentration values for lipid molecular species using class-specific internal standards (e.g., SPLASH Lipidomix species) with known concentrations, as implemented in the Lipid Data Quantification utility of LipidOne 2.0.

## When to use

Use when the input is a lipidomic peak-area/peak-height matrix whose rows include internal standard species and a companion standards table (Standard/Class/Conc.) exists, and the goal is a per-species concentration matrix.

## When NOT to use

- Input data are already concentration values rather than raw peak areas/heights — re-applying conversion factors would double-normalize
- The dataset lacks internal standard rows or a standards table (no Standard/Class/Conc. mapping), so class-specific conversion factors cannot be constructed
- The goal is downstream statistical analysis (mono-/multivariate tests, biomarker identification) on already-quantified data — use LipidOne 2.0's data-mining functions instead of the quantification utility

## Inputs

- Peak-area or peak-height data matrix of lipid molecular species (rows = species incl. internal standard rows such as SPLASH Lipidomix; columns = samples)
- 'standard.txt' standards table with columns Standard/Class/Conc. giving internal standard names, lipid-class assignment, and known concentration

## Outputs

- Concentration matrix written to quantified_matrix.csv (one row per unique molecular species, internal standard rows removed, duplicates consolidated by summation)

## How to apply

Load the peak-area/peak-height data matrix (including internal standard rows such as SPLASH Lipidomix species) and the 'standard.txt' table with columns Standard/Class/Conc. as pandas or R data frames. For each internal standard, combine its known concentration (Conc. column) with its lipid-class assignment (Class column) to build a class-specific conversion factor, then transform the peak areas of all molecular species belonging to the matching lipid class into concentrations. After quantification, automatically exclude rows containing standards from the data matrix so they do not appear as analytes. Detect molecular species that appear in multiple rows and consolidate duplicates into a single entry per species by summation of their quantified amounts. Write the resulting concentration matrix to quantified_matrix.csv. The class-level mapping is the key decision point: a species without a matching class in standard.txt cannot be quantified, so every species label must be resolvable to a class before conversion.

## Related tools

- **LipidOne 2.0** (Web platform hosting the Lipid Data Quantification utility (Utility section) that performs the peak-area-to-concentration conversion and post-quantification matrix cleanup) — https://lipidone.eu
- **R** (Back-end scripting language and reproduction environment for the quantification workflow (loading data frames, building class-specific conversion factors, filtering standard rows, consolidating duplicates))

## Examples

```
quant = df[~df['Species'].isin(standards['Standard'])].groupby('Species', as_index=False).sum(); quant.to_csv('quantified_matrix.csv', index=False)
```

## Evaluation signals

- No internal standard rows (e.g., SPLASH Lipidomix species) remain in the quantified matrix
- Every molecular species in the output is mapped to a lipid class via standard.txt (no unquantifiable/unmapped species persist)
- No duplicate molecular species labels persist in quantified_matrix.csv — each species appears as a single row
- Duplicate species entries, when present pre-consolidation, have quantified amounts equal to the sum of their constituent rows
- Species belonging to classes with an internal standard in standard.txt carry concentration values consistent with that class's conversion factor

## Limitations

- Quantification is class-specific: species in a lipid class without a matching internal standard in standard.txt cannot be converted to concentration
- Consolidation assumes summation is the appropriate merge for duplicate species rows; other aggregation strategies are not described
- The LipidOne 2.0 source code itself is not explicitly stated to be publicly hosted in the article section text (only the older Perl desktop LipidOne repo README was provided, which does not document the quantification utility)

## Evidence

- [other] Class-specific conversion via standards table: [evidence span withheld — anchor status: not_found]
- [other] Post-quantification filtering and consolidation: [evidence span withheld — anchor status: not_found]
- [abstract] Three structural levels supported: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [readme] Tool scope from repository README: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
