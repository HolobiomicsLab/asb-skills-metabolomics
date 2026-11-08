---
name: lipid-chain-length-unsaturation-inference
description: Use when lipid molecule name strings (e.g. from Metabolomics Workbench
  study ST001111, Skyline CSV exports, or numerical matrices) need class/chain/unsaturation
  annotations, or when entries such as full-chemical-name ceramides fail to parse
  and must be renamed via RegEx before annotation.
when_to_use_negative:
- The goal is downstream statistical analysis (differential analysis, PCA/OPLS, LSEA)
  on data whose annotations are already correctly parsed — this skill covers only
  the name-parsing/annotation step
- Molecule identifiers are not lipid names in lipidr-supported nomenclature and no
  RegEx renaming mapping can be supplied, so lipidnames_pattern matching will flag
  everything as unparseable
- Input is a numerical feature table with arbitrary feature IDs (not lipid names),
  where name-based inference cannot apply
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: R/Bioconductor package providing lipidnames_pattern regex rules, automatic
    class/chain inference on import, and update_molecule_names for RegEx-based renaming
    of unparseable names
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment; lipidr represents datasets as a LipidomicsExperiment
    extending SummarizedExperiment
- name: Metabolomics Workbench API
  role: Source of public lipidomics datasets (e.g. ST001111) whose molecule names
    are parsed for annotations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:f06656cab3e680fdc2417fc995c3ec703430ec7a35886a70ef7a989727ab7848
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lipid-chain-length-unsaturation-inference

## Summary

Infers per-molecule lipid class, total chain length, and unsaturation (double-bond) annotations by parsing lipid molecule name strings with lipidr's lipidnames_pattern regex rules, renaming non-conforming entries (e.g. full-chemical-name ceramides) to shorthand via update_molecule_names. This is the annotation layer that powers lipidr's downstream filtering and LSEA by class, chain length, or unsaturation.

## When to use

Use when lipid molecule name strings (e.g. from Metabolomics Workbench study ST001111, Skyline CSV exports, or numerical matrices) need class/chain/unsaturation annotations, or when entries such as full-chemical-name ceramides fail to parse and must be renamed via RegEx before annotation.

## When NOT to use

- The goal is downstream statistical analysis (differential analysis, PCA/OPLS, LSEA) on data whose annotations are already correctly parsed — this skill covers only the name-parsing/annotation step
- Molecule identifiers are not lipid names in lipidr-supported nomenclature and no RegEx renaming mapping can be supplied, so lipidnames_pattern matching will flag everything as unparseable
- Input is a numerical feature table with arbitrary feature IDs (not lipid names), where name-based inference cannot apply

## Inputs

- Lipid molecule name strings from a numerical matrix (first column of a lipids-by-samples CSV)
- Skyline results exported as CSV (read_skyline)
- Metabolomics Workbench study files (e.g. ST001111)
- Sample annotation / groups table (sample names matching the data file)
- RegEx mapping rules for renaming non-conforming molecule names (e.g. full-chemical-name ceramides to shorthand)

## Outputs

- LipidomicsExperiment object with per-molecule class and chain annotations inferred from names
- Molecule-to-annotation mapping table (lipid_annotations.csv) with class, total chain length, and unsaturation columns
- Renamed molecule list where full-chemical-name entries have been converted to shorthand notation

## How to apply

Load the lipid molecule name strings from your input (read_skyline CSV, Metabolomics Workbench file, or numerical matrix) into R as a LipidomicsExperiment. Apply lipidr's lipidnames_pattern regex rules to each name string to extract lipid class, total chain length, and unsaturation (double-bond) information; lipidr performs this automatically on import. For names that do not match supported patterns — the vignette's example is full-chemical-name ceramides from ST001111 — construct RegEx mappings and apply update_molecule_names to convert them to shorthand notation so annotations can be inferred. Assemble the parsed class/chain/unsaturation fields into a molecule-to-annotation mapping (e.g. lipid_annotations.csv) so downstream steps such as %CV filtering and LSEA can group lipids by class, total chain length, or unsaturation pattern.

## Related tools

- **lipidr** (R/Bioconductor package providing lipidnames_pattern regex rules, automatic class/chain inference on import, and update_molecule_names for RegEx-based renaming of unparseable names) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment; lipidr represents datasets as a LipidomicsExperiment extending SummarizedExperiment)
- **Metabolomics Workbench API** (Source of public lipidomics datasets (e.g. ST001111) whose molecule names are parsed for annotations)

## Examples

```
d <- as_lipidomics_experiment(read.csv("data_matrix.csv")); d <- add_sample_annotation(d, "data_clin.csv")
```

## Evaluation signals

- The molecule-to-annotation mapping table (lipid_annotations.csv) exists and is readable as a CSV
- Every molecule row carries non-empty class and chain annotations after lipidnames_pattern parsing
- Full-chemical-name ceramide entries no longer appear; they have been converted to shorthand notation via update_molecule_names
- Names flagged by lipidnames_pattern as unmatched should be an explicit, documented set — the vignette case is full-chemical-name ceramides, which should become parseable after RegEx renaming
- The resulting LipidomicsExperiment supports grouping lipids by class, total chain length, or unsaturation pattern (e.g. for LSEA), confirming annotations were attached correctly

## Limitations

- Name parsing only works for molecule names matching lipidr's supported patterns; non-conforming names are flagged rather than annotated
- Unparseable molecules require user-supplied RegEx renaming (update_molecule_names); lipidr will not guess mappings for full-chemical-name lipids
- Sample names must be identical in the data matrix and the sample annotation file when building the LipidomicsExperiment

## Evidence

- [readme] LipidomicsExperiment object: [evidence span withheld — anchor status: external_doc]
- [readme] Lipid names must be in the first column: [evidence span withheld — anchor status: external_doc]
