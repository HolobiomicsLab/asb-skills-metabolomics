---
name: lipidomics-metadata-annotation
description: Use when a lipidomics matrix (numerical matrix, Skyline CSV export, or
  Metabolomics Workbench file) contains lipid molecule name strings that need class,
  chain-length, and double-bond annotations, or when some names fail pattern matching.
when_to_use_negative:
- The input is not a lipidomics dataset with lipid name strings (lipidr's automatic
  class/chain inference depends on parseable lipid names).
- Molecule names are already standardized shorthand with class/chain/unsaturation
  encoded and no re-annotation is needed.
- The analysis goal is generic metabolite identification from spectra rather than
  annotation of lipid molecule names.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: R/Bioconductor package providing lipidnames_pattern regex rules, annotate_lipids
    parsing, and update_molecule_names RegEx renaming to derive class/chain/unsaturation
    annotations
  repo: github.com/ahmohamed/lipidr
- name: R
  role: Runtime environment in which lipidr annotation functions are executed and
    the mapping CSV is written
- name: Metabolomics Workbench API
  role: Source of public lipidomics datasets (e.g. ST001111) whose molecule names
    are parsed and, where needed, renamed before annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:e6287c4b08b9abcfdb9bad369669c270aadbad3ac76921bbb0b00f3a27ec0d92
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R
  - lipidr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipidomics-metadata-annotation

## Summary

Parses lipid molecule name strings with lipidr's lipidnames_pattern rules to derive lipid class, total chain length, and unsaturation annotations, flagging unparseable names and renaming full-chemical-name entries (e.g. ceramides) via RegEx so annotations are correctly inferred.

## When to use

Use when a lipidomics matrix (numerical matrix, Skyline CSV export, or Metabolomics Workbench file) contains lipid molecule name strings that need class, chain-length, and double-bond annotations, or when some names fail pattern matching.

## When NOT to use

- The input is not a lipidomics dataset with lipid name strings (lipidr's automatic class/chain inference depends on parseable lipid names).
- Molecule names are already standardized shorthand with class/chain/unsaturation encoded and no re-annotation is needed.
- The analysis goal is generic metabolite identification from spectra rather than annotation of lipid molecule names.

## Inputs

- Lipid molecule name strings from a numerical matrix (lipid names in first column, e.g. brca_matrix.csv style input)
- Skyline export CSV (e.g. A1_data.csv style input) containing molecule names
- Metabolomics Workbench study files (e.g. ST001111) including full-chemical-name ceramide entries
- lipidr lipidnames_pattern regex rule set

## Outputs

- Molecule-to-annotation mapping table (lipid_annotations.csv) with lipid class, total chain length, and unsaturation per molecule
- LipidomicsExperiment object with inferred lipid class and chain information attached to molecule names
- List/set of molecule names that failed pattern matching and require RegEx renaming

## How to apply

Load the lipid molecule name strings (lipidr expects lipid names in the first column of a numerical matrix, or within a Skyline export CSV) and match each against lipidr's lipidnames_pattern regex rules to extract lipid class, total chain length, and unsaturation (double-bond) information. Names that do not match any supported pattern are flagged rather than silently dropped, so inspect these before proceeding to downstream steps such as LSEA, which relies on class, total chain length, and unsaturation groupings. For unparseable entries — for example, full-chemical-name ceramides from Metabolomics Workbench study ST001111 — rewrite them to shorthand notation using lipidr's update_molecule_names with an appropriate RegEx, then re-run the annotation. Assemble the parsed class/chain/unsaturation fields into a molecule-to-annotation mapping table and write it as lipid_annotations.csv. This parsing is what lipidr performs automatically when it imports data into a LipidomicsExperiment and infers lipid class and chain information from lipid names.

## Related tools

- **lipidr** (R/Bioconductor package providing lipidnames_pattern regex rules, annotate_lipids parsing, and update_molecule_names RegEx renaming to derive class/chain/unsaturation annotations) — github.com/ahmohamed/lipidr
- **R** (Runtime environment in which lipidr annotation functions are executed and the mapping CSV is written)
- **Metabolomics Workbench API** (Source of public lipidomics datasets (e.g. ST001111) whose molecule names are parsed and, where needed, renamed before annotation)

## Examples

```
d <- as_lipidomics_experiment(read.csv("data_matrix.csv")); d <- annotate_lipids(d); d <- update_molecule_names(d, pattern = "N-...", replace = "Cer ...", by = "molecule")
```

## Evaluation signals

- lipid_annotations.csv exists and is readable as CSV with one row per molecule.
- Every molecule row carries a non-empty class annotation and a non-empty total chain length annotation.
- Full-chemical-name ceramide entries have been converted to shorthand notation (via update_molecule_names) and now yield correct class/chain/unsaturation annotations.
- No molecule silently lacks annotation: unparseable names are flagged rather than dropped.
- Downstream LSEA groupings by lipid class, total chain length, and unsaturation are computable from the annotated table.

## Limitations

- Molecule names must match lipidr's supported lipidnames_pattern rules; non-conforming names are flagged and not annotated until renamed.
- Full-chemical-name ceramides (e.g. from Metabolomics Workbench study ST001111) fail standard parsing and require manual RegEx renaming with update_molecule_names before annotation can be inferred.
- Automatic class/chain inference depends on lipid names being present in the input; inputs without lipid name strings cannot be annotated.

## Evidence

- [readme] Lipid names expected in first column: [evidence span withheld — anchor status: external_doc]
