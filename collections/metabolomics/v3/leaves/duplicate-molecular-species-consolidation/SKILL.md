---
name: duplicate-molecular-species-consolidation
description: Use when a lipidomics concentration matrix (post internal-standard quantification)
  still contains SPLASH Lipidomix/standard rows and multiple rows for the same molecular
  species label that must be merged before downstream analysis.
when_to_use_negative:
- The matrix is still raw peak areas and internal standards have not yet been used
  to compute class-specific conversion factors — deduplicate only after quantification
  to concentration
- Every molecular species already appears exactly once and no standard rows remain
  — the matrix is already in the post-cleanup state
- The analysis operates at the lipid building-block (acyl/alkyl chain) level rather
  than the molecular-species level, where duplicate-species merging does not apply
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidOne 2.0
  role: Web tool whose Utility 'Lipid Data Quantification' function performs the standard-based
    conversion, standard-row removal, and duplicate-species consolidation; this skill
    reimplements that post-quantification cleanup
  repo: https://lipidone.eu
- name: R
  role: Back-end language (R data frames) used to implement the standard lookup, row
    filtering, and duplicate summation logic
- name: LipidOne (desktop)
  role: Predecessor standalone tool; its GitHub repository (Perl/Windows, FreeBSD
    license) provides the original codebase context, though it does not document the
    2.0 quantification cleanup
  repo: https://github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:f44a8808787bd6ea7661cb2666caabdaa22dda03520644324b3654985d74a605
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne (desktop)
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# duplicate-molecular-species-consolidation

## Summary

Post-quantification cleanup step from the LipidOne 2.0 Utility 'Lipid Data Quantification' function: after internal-standard-based conversion of peak areas/heights into concentrations, internal standard rows are dropped and duplicate rows for the same molecular species are summed into a single entry per species.

## When to use

Use when a lipidomics concentration matrix (post internal-standard quantification) still contains SPLASH Lipidomix/standard rows and multiple rows for the same molecular species label that must be merged before downstream analysis.

## When NOT to use

- The matrix is still raw peak areas and internal standards have not yet been used to compute class-specific conversion factors — deduplicate only after quantification to concentration
- Every molecular species already appears exactly once and no standard rows remain — the matrix is already in the post-cleanup state
- The analysis operates at the lipid building-block (acyl/alkyl chain) level rather than the molecular-species level, where duplicate-species merging does not apply

## Inputs

- Peak-area or peak-height data matrix (CSV), including internal standard rows (e.g. SPLASH Lipidomix species)
- standard.txt lookup table with columns Standard / Class / Conc. giving each internal standard's concentration and lipid class

## Outputs

- quantified_matrix.csv: concentration matrix with no internal standard rows and exactly one row per molecular species (duplicates merged by summation)

## How to apply

Load the peak-area/peak-height matrix (including internal standard rows such as SPLASH Lipidomix species) and the 'standard.txt' table with Standard/Class/Conc. columns into pandas or R data frames. Build class-specific conversion factors from each standard's known Conc. and its Class assignment, converting peak areas of all species in the matching lipid class to concentrations. Then drop the internal standard rows so they do not appear as analytes, and detect molecular species occurring in multiple rows, consolidating them by summation of their quantified amounts into one entry per species. Finally write the cleaned concentration matrix to quantified_matrix.csv. The rationale is that a lipidomic dataset is explored at the class, molecular species, and building-block levels in LipidOne 2.0, so the species-level matrix must be unique-per-species and free of standard rows before statistics.

## Related tools

- **LipidOne 2.0** (Web tool whose Utility 'Lipid Data Quantification' function performs the standard-based conversion, standard-row removal, and duplicate-species consolidation; this skill reimplements that post-quantification cleanup) — https://lipidone.eu
- **R** (Back-end language (R data frames) used to implement the standard lookup, row filtering, and duplicate summation logic)
- **LipidOne (desktop)** (Predecessor standalone tool; its GitHub repository (Perl/Windows, FreeBSD license) provides the original codebase context, though it does not document the 2.0 quantification cleanup) — https://github.com/matteogiulietti/LipidOne

## Examples

```
quant = pd.read_csv('peak_areas.csv'); quant = quant[~quant.Species.isin(standards.Standard)]; quant = quant.groupby('Species', as_index=False).sum(); quant.to_csv('quantified_matrix.csv', index=False)
```

## Evaluation signals

- No internal standard rows (e.g. SPLASH Lipidomix species listed in standard.txt) remain in quantified_matrix.csv
- Every molecular species in the output is mapped to a lipid class via the standard.txt Class column
- No duplicate molecular species labels persist in the output matrix — each label appears exactly once
- The quantified amount for a consolidated species equals the sum of the amounts of its duplicate rows before merging
- Output schema matches the input matrix schema (same sample columns), with only rows removed/merged

## Limitations

- The GitHub README documents the older standalone LipidOne written in Perl for Windows 64-bit (Prima/Gnuplot GUI), not the R-backed Lipid One 2.0 web tool, so repo docs give no direct guidance on this cleanup step
- Whether the Lipid One 2.0 source code itself is publicly hosted is not stated; example datasets are only accessible through the web platform rather than a persistent repository accession
- Consolidation by summation assumes duplicate rows are true redundant measurements of the same molecular species; the article does not state how ambiguous or class-mismatched duplicates are resolved

## Evidence

- [abstract] LipidOne 2.0 web tool context: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [abstract] Three structural levels requiring clean species-level matrix: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [abstract] Class filtering flexibility after cleanup: "The tool's flexibility empowers users to seamlessly include or exclude experimental groups and lipid classes at any stage of the analysis."
- [readme] README on LipidOne's purpose (older desktop version): "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
