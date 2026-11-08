---
name: lipid-name-regex-parsing
description: Use when lipidomics molecule names (e.g. from Metabolomics Workbench
  study ST001111 or Skyline exports) need class, chain-length, and unsaturation annotations
  inferred from name strings, and some names fail pattern matching.
when_to_use_negative:
- The input already carries explicit lipid class/chain columns or uses IUPAC/LIPID
  MAPS identifiers resolvable by a database rather than name parsing
- The task is downstream analysis (normalization, differential analysis, LSEA) on
  an already-annotated LipidomicsExperiment
- Molecule names are isotopologue/fragment-level entries from Skyline internals rather
  than lipid names at the row level
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: Provides lipidnames_pattern regex rules and update_molecule_names to parse
    names and rename unparseable molecules before inferring annotations
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Execution environment in which lipidr's parsing and renaming functions are
    run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:27f6c6d42b5c5ca373d04e4de19553a37302796984fa2f60db7c16de8696e5c1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# lipid-name-regex-parsing

## Summary

Parses lipid molecule name strings against lipidr's lipidnames_pattern regex rules to extract lipid class, total chain length, and unsaturation (double-bond) annotations, using update_molecule_names with RegEx to convert non-conforming names such as full-chemical-name ceramides into shorthand before annotation.

## When to use

Use when lipidomics molecule names (e.g. from Metabolomics Workbench study ST001111 or Skyline exports) need class, chain-length, and unsaturation annotations inferred from name strings, and some names fail pattern matching.

## When NOT to use

- The input already carries explicit lipid class/chain columns or uses IUPAC/LIPID MAPS identifiers resolvable by a database rather than name parsing
- The task is downstream analysis (normalization, differential analysis, LSEA) on an already-annotated LipidomicsExperiment
- Molecule names are isotopologue/fragment-level entries from Skyline internals rather than lipid names at the row level

## Inputs

- Lipid molecule name strings (first column of a numerical matrix CSV or Skyline-exported CSV)
- lipidnames_pattern regex rules bundled with lipidr
- Mapping RegEx patterns for renaming non-parseable molecules (e.g. full-chemical-name ceramides)

## Outputs

- Molecule-to-annotation mapping table (lipid_annotations.csv) with class, total chain length, and unsaturation per molecule
- List of molecule names that failed pattern matching and were flagged as unparseable

## How to apply

Load the lipid molecule name strings (for example from Metabolomics Workbench study ST001111) into R and apply lipidr's lipidnames_pattern regex rules to each name to extract lipid class, total chain length, and unsaturation information. Names that do not match any supported pattern are flagged rather than annotated. For flagged molecules such as full-chemical-name ceramides, use lipidr's update_molecule_names with a RegEx to rename them to shorthand equivalents (e.g. Cer d18:1/16:0 style) so annotations can be correctly inferred. Assemble the parsed class/chain/unsaturation fields into a molecule-to-annotation mapping table (lipid_annotations.csv) and validate that every row carries non-empty class and chain annotations. This mirrors lipidr's import behavior of 'automatically inferring lipid class and chain information from lipid names'.

## Related tools

- **lipidr** (Provides lipidnames_pattern regex rules and update_molecule_names to parse names and rename unparseable molecules before inferring annotations) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Execution environment in which lipidr's parsing and renaming functions are run)

## Examples

```
d <- as_lipidomics_experiment(read.csv("data_matrix.csv")); d <- update_molecule_names(d, old_names, new_names)
```

## Evaluation signals

- lipid_annotations.csv exists and is readable as a CSV
- Every molecule row carries a non-empty lipid class annotation and a non-empty chain-length annotation
- Ceramide entries originally given as full chemical names have been converted to shorthand notation after update_molecule_names
- No molecule remains unannotated except those legitimately flagged as unmatched by lipidnames_pattern; each flagged name is explainable
- Unsaturation (double-bond) counts parsed from names match the numbers in the shorthand names (e.g. total double bonds consistent with class/chain strings)

## Limitations

- Molecule names that do not match supported lipidnames_pattern patterns are flagged as unparseable and cannot be annotated until renamed; the vignette's workaround is RegEx renaming via update_molecule_names (e.g. full-chemical-name ceramides)
- Automatic annotation depends on the naming convention used by the data source; Skyline exports or Workbench files with non-standard names will fail pattern matching

## Evidence

- [readme] README: LipidomicsExperiment object structure: [evidence span withheld — anchor status: not_found]
- [readme] README: lipid names in first column of numerical matrix input: [evidence span withheld — anchor status: external_doc]
