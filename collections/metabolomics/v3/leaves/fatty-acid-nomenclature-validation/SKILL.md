---
name: fatty-acid-nomenclature-validation
description: Use when building or auditing in silico lipid fragmentation libraries
  to verify fatty acid combination lists, lipid species totals, distinct lipid type
  counts, and permutation redundancy (e.g. 18:0_20:0 vs 20:0_18:0).
when_to_use_negative:
- Your goal is validating experimental lipid identifications on a processed feature
  table — this skill only audits in silico library composition, not MS/MS matches.
- Input is MS/MS .ms2 files or feature tables rather than lipid library .csv files
  — nomenclature validation operates on the library csvs.
- You expect the current GitHub distribution's counts (>500,000 species, >60 types)
  rather than the 2017 paper-era counts (>250,000 species, 56 types) — version matters.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Used to generate fatty acid combination lists, load and tally unique lipid
    species in each library .csv, count distinct lipid types, and scan class/adduct
    labels
- name: LipidMatch
  role: Source of the in silico fragmentation libraries being validated, distributed
    as 2017_6_14_LipidMatch_Distribution.zip with libraries in .csv format
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:4ede9faad1636bbde289d1d322a5e5422b5d682daf9a1977642440811af4f86d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - LipidMatch
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fatty-acid-nomenclature-validation

## Summary

Validates the fatty acid and lipid-species composition of in silico lipid fragmentation libraries, e.g. enumerating LipidMatch's 39 endogenous and 214 oxidized fatty acids and the >250,000 species across 56 lipid types while removing redundant acyl-chain permutations.

## When to use

Use when building or auditing in silico lipid fragmentation libraries to verify fatty acid combination lists, lipid species totals, distinct lipid type counts, and permutation redundancy (e.g. 18:0_20:0 vs 20:0_18:0).

## When NOT to use

- Your goal is validating experimental lipid identifications on a processed feature table — this skill only audits in silico library composition, not MS/MS matches.
- Input is MS/MS .ms2 files or feature tables rather than lipid library .csv files — nomenclature validation operates on the library csvs.
- You expect the current GitHub distribution's counts (>500,000 species, >60 types) rather than the 2017 paper-era counts (>250,000 species, 56 types) — version matters.

## Inputs

- LipidMatch in silico fragmentation libraries in .csv format (from 2017_6_14_LipidMatch_Distribution.zip, Additional file 1 or http://secim.ufl.edu/secim-tools/)
- List of unsaturated fatty acids from LIPID MAPS for generating short-chain oxidized fatty acids
- R script generating possible fatty acid combinations for acyl linkages

## Outputs

- library_composition_report.csv with total species count, per-lipid-type counts, fatty acid list sizes, and unique-class presence flags
- Fatty acid combination lists: 39 endogenous fatty acids and 214 potential oxidized fatty acids (126 long-chain O/OH/OOH + 88 short-chain CHO/COOH)

## How to apply

Use an R script to generate fatty acid combination lists for acyl linkages, splitting oxidized fatty acids into long-chain O/OH/OOH additions (126) and short-chain CHO/COOH cleavage products (88, derived by cleaving unsaturated fatty acids from LIPID MAPS and adding a terminal aldehyde or carboxylic acid group). Exclude redundant head-to-tail permutations such as 18:0_20:0 vs 20:0_18:0 so each unique combination is counted once. Extract the library distribution zip (2017_6_14_LipidMatch_Distribution.zip), load each library .csv in R, tally unique lipid species per file, and sum across libraries as a lower-bound species count (paper-era distribution: over 250,000). Count distinct lipid types requiring an exact match to 56, applying counting conventions: each plasmenyl/plasmanyl ether pair as two types and oxidized lipids across classes as one type. Scan class labels and adduct annotations to confirm unique libraries (oxidized lipids, bile acids, sphingosines, ammoniated cardiolipins) and write counts plus presence flags to a report.

## Related tools

- **R** (Used to generate fatty acid combination lists, load and tally unique lipid species in each library .csv, count distinct lipid types, and scan class/adduct labels)
- **LipidMatch** (Source of the in silico fragmentation libraries being validated, distributed as 2017_6_14_LipidMatch_Distribution.zip with libraries in .csv format) — https://github.com/GarrettLab-UF/LipidMatch

## Examples

```
libs <- lapply(list.files("2017_6_14_LipidMatch_Distribution", pattern="\\.csv$", full.names=TRUE), read.csv); sum(sapply(libs, nrow))
```

## Evaluation signals

- Total unique lipid species summed across all library .csv files is a lower bound of over 250,000 for the 2017 distribution (the current README version cites over 500,000, so pin the version before comparing).
- Distinct lipid type count matches exactly 56 under the stated conventions (plasmenyl/plasmanyl ether pairs counted as two types; oxidized lipids across classes as one type).
- Endogenous fatty acid list size is exactly 39; oxidized list is exactly 214, decomposing as 126 long-chain O/OH/OOH additions plus 88 short-chain CHO/COOH cleavage products.
- Redundant permutations such as 18:0_20:0 vs 20:0_18:0 are excluded from combination counts (no duplicate unordered acyl pairs).
- Class/adduct label scan flags the unique libraries: oxidized lipids, bile acids, sphingosines, and cardiolipins as ammonium adducts are present.

## Limitations

- Composition counts are pinned to a specific software version: the 2017 paper reports over 250,000 species across 56 types, while the current README reports over 500,000 species across over 60 types.
- The species total is evaluated as a lower bound, not an exact count.
- Lipid type totals depend on counting conventions (ether plasmenyl/plasmanyl pairs, oxidized lipids collapsed across classes), which are judgment calls that change the result.
- Key artifacts (software zip, lipid libraries in .csv format) are provided only as additional files / a download link, not archived with DOIs in a public repository, complicating reproducibility of the exact count.

## Evidence

- [methods] an R script was used to generate a list of possible fatty acid combinations for acyl: "an R script was used to generate a list of possible fatty acid combinations for acyl"
- [methods] 88 short-chain oxidized fatty acids from LIPID MAPS cleavage: "A list of 88 potential short chain oxidized fatty acids were generated by cleavage of unsaturated fatty acids contained in LIPID MAPS"
- [methods] 56 lipid types coverage claim: "With 56 lipid types, LipidMatch in silico libraries cover the greatest number of lipid types of any open source software to date"
- [readme] README version composition counts differ: "in-silico fragmentation libraries of over 500,000 lipid species across over 60 lipid types"
- [readme] libraries distributed as csv files: "The link at the bottom of this page contains a manual, lipid libraries in .csv format"
