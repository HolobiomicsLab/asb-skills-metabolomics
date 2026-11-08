---
name: standard-row-filtering
description: Use when a lipidomic peak-area or peak-height matrix contains internal
  standard rows (e.g. SPLASH Lipidomix species) that must be removed after class-specific
  conversion to concentration, and species duplicates consolidated.
when_to_use_negative:
- The input matrix contains no internal standard rows, so no class-specific conversion
  factors from standard.txt can be built
- Peak areas have already been converted to concentrations and species duplicates
  already consolidated
- The goal is a downstream LipidOne 2.0 analysis (pathway analysis, biomarker identification)
  on an already-quantified matrix rather than reconstructing the quantification step
  itself
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: LipidOne 2.0
  role: Web platform whose Utility > Lipid Data Quantification function performs the
    standard-row exclusion and species consolidation; this skill reconstructs that
    behavior outside the web tool
  repo: https://lipidone.eu
- name: R
  role: Implementation environment for the reconstruction (pandas/R data frames) to
    load the matrix and standard.txt, apply class-specific conversion factors, filter
    standard rows, sum duplicates, and export quantified_matrix.csv
  repo: https://github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:a4abd9df231c5047422518b03076c200af056214adaf367375aed99dfcbbce4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# standard-row-filtering

## Summary

Post-quantification cleanup from LipidOne 2.0's Utility section: after converting lipid peak areas/heights to concentrations using internal standards, the internal standard rows are excluded and duplicate molecular species rows are summed into one entry per species.

## When to use

Use when a lipidomic peak-area or peak-height matrix contains internal standard rows (e.g. SPLASH Lipidomix species) that must be removed after class-specific conversion to concentration, and species duplicates consolidated.

## When NOT to use

- The input matrix contains no internal standard rows, so no class-specific conversion factors from standard.txt can be built
- Peak areas have already been converted to concentrations and species duplicates already consolidated
- The goal is a downstream LipidOne 2.0 analysis (pathway analysis, biomarker identification) on an already-quantified matrix rather than reconstructing the quantification step itself

## Inputs

- Peak-area or peak-height data matrix including internal standard rows (e.g. SPLASH Lipidomix species)
- standard.txt lookup table with columns Standard/Class/Conc. giving known concentration and lipid-class assignment per internal standard

## Outputs

- quantified_matrix.csv — concentration matrix with internal standard rows removed and one consolidated row per molecular species

## How to apply

Load the peak-area/peak-height matrix (with internal standard rows such as SPLASH Lipidomix) and the 'standard.txt' table with Standard/Class/Conc. columns into pandas or R data frames. For each internal standard, use its known Conc. value and Class assignment to build a class-specific conversion factor, and transform peak areas of all species in the matching class into concentrations. Then remove the internal standard rows so they do not appear as analytes, and detect molecular species present in multiple rows, consolidating them by summation of their quantified amounts into one row per species. Write the cleaned concentration matrix to quantified_matrix.csv; the rationale is that standard rows and duplicate species labels would otherwise contaminate downstream class/species-level statistics in LipidOne 2.0.

## Related tools

- **LipidOne 2.0** (Web platform whose Utility > Lipid Data Quantification function performs the standard-row exclusion and species consolidation; this skill reconstructs that behavior outside the web tool) — https://lipidone.eu
- **R** (Implementation environment for the reconstruction (pandas/R data frames) to load the matrix and standard.txt, apply class-specific conversion factors, filter standard rows, sum duplicates, and export quantified_matrix.csv) — https://github.com/matteogiulietti/LipidOne

## Examples

```
m=pd.read_csv('matrix.csv',index_col=0); s=pd.read_csv('standard.txt',sep='\t'); m[~m.index.isin(s['Standard'])].groupby(level=0).sum().to_csv('quantified_matrix.csv')
```

## Evaluation signals

- No internal standard rows remain in the output quantified_matrix.csv
- Every molecular species is mapped to a class via the standard.txt Standard/Class/Conc. table
- No duplicate molecular species labels persist in the output matrix
- Consolidated amounts for previously duplicated species equal the summation of their per-row quantified values
- Concentrations for each species are derivable from its class standard's Conc. value via the class-specific conversion factor

## Limitations

- Conversion depends on every lipid class having a matching internal standard in standard.txt; classes without a standard cannot be converted to concentration
- The article section text does not detail the exact conversion formula or numeric thresholds — behavior is reconstructed from task-card workflow descriptions
- The GitHub README describes the older desktop LipidOne written in Perl for Microsoft Windows only, not the R-backed LipidOne 2.0 web tool where this function lives

## Evidence

- [other] Post-quantification cleanup in Lipid Data Quantification: [evidence span withheld — anchor status: not_found]
- [abstract] LipidOne 2.0 as web tool: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [discussion] Example dataset access for validation runs: [evidence span withheld — anchor status: not_found]
