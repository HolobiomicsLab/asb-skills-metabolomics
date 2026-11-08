---
name: ceramide-nomenclature-mapping
description: Use when lipid molecule name strings (e.g. from Metabolomics Workbench
  study ST001111 or Skyline exports) need class/chain-length/double-bond annotations,
  or when full-chemical-name ceramide entries fail name parsing and must be mapped
  to shorthand notation.
when_to_use_negative:
- The dataset is already a lipidr LipidomicsExperiment with class/chain annotations
  inferred at import time — re-parsing names would be redundant
- The goal is downstream statistics (normalization, univariate analysis, LSEA) on
  already-annotated lipids rather than name-to-annotation mapping itself
- Molecule names fall outside lipidr's supported lipid patterns and no RegEx renaming
  (update_molecule_names) can be supplied — entries will simply be flagged as unparseable
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: Supplies lipidnames_pattern regex rules, the annotate_lipids parsing mechanism,
    and update_molecule_names for RegEx renaming of full-chemical-name ceramides to
    shorthand
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Runtime environment in which lipidr's name-parsing and renaming functions
    are executed
- name: Metabolomics Workbench API
  role: Source of public lipidomics datasets (e.g. ST001111) whose molecule name strings
    serve as parsing input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:e98013090a00cf278c306649bf0533b3c452b33784748cdf868229cb3d53222d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# ceramide-nomenclature-mapping

## Summary

Parses lipid molecule name strings against lipidr's lipidnames_pattern regex rules to infer lipid class, total chain length, and unsaturation, renaming unparseable full-chemical-name ceramides to shorthand via update_molecule_names so annotations can be correctly inferred.

## When to use

Use when lipid molecule name strings (e.g. from Metabolomics Workbench study ST001111 or Skyline exports) need class/chain-length/double-bond annotations, or when full-chemical-name ceramide entries fail name parsing and must be mapped to shorthand notation.

## When NOT to use

- The dataset is already a lipidr LipidomicsExperiment with class/chain annotations inferred at import time — re-parsing names would be redundant
- The goal is downstream statistics (normalization, univariate analysis, LSEA) on already-annotated lipids rather than name-to-annotation mapping itself
- Molecule names fall outside lipidr's supported lipid patterns and no RegEx renaming (update_molecule_names) can be supplied — entries will simply be flagged as unparseable

## Inputs

- Lipid molecule name strings (rows of a numerical matrix CSV, Skyline-exported CSV, or Metabolomics Workbench study file, e.g. ST001111)
- lipidr's lipidnames_pattern regex rules (shipped with the lipidr R/Bioconductor package)
- RegEx substitution patterns for renaming full-chemical-name ceramide entries to shorthand

## Outputs

- Molecule-to-annotation mapping table (lipid_annotations.csv) with lipid class, total chain length, and unsaturation columns per molecule
- Renamed molecule list with full-chemical-name ceramides converted to shorthand notation

## How to apply

Load the lipid molecule name strings (lipidr accepts numerical matrices, Skyline CSV exports, and Metabolomics Workbench files, automatically inferring lipid class and chain information from lipid names). Apply lipidr's lipidnames_pattern regex rules to each name string to extract the lipid class, total chain length, and unsaturation (double-bond) information; names that do not match any supported pattern are flagged rather than silently annotated. For unparseable entries such as full-chemical-name ceramides, rename them to their shorthand equivalents with lipidr's update_molecule_names using RegEx patterns, then re-run annotation. Assemble the parsed class/chain/unsaturation fields into a molecule-to-annotation mapping table (e.g. lipid_annotations.csv) and validate that every molecule row carries non-empty class and chain annotations and that ceramide entries appear in shorthand.

## Related tools

- **lipidr** (Supplies lipidnames_pattern regex rules, the annotate_lipids parsing mechanism, and update_molecule_names for RegEx renaming of full-chemical-name ceramides to shorthand) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Runtime environment in which lipidr's name-parsing and renaming functions are executed)
- **Metabolomics Workbench API** (Source of public lipidomics datasets (e.g. ST001111) whose molecule name strings serve as parsing input)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- annotate_lipids(d); d <- update_molecule_names(d, ceramide_regex_table, "Molecule")
```

## Evaluation signals

- lipid_annotations.csv exists and is readable as a CSV with one row per molecule
- Every molecule row carries non-empty lipid class and chain-length annotations
- All full-chemical-name ceramide entries have been converted to shorthand notation after update_molecule_names
- Unsaturation (double-bond) values in the table are consistent with the numerals embedded in each molecule name
- Remaining unmatched names are explicitly flagged rather than silently assigned empty annotations

## Limitations

- Name parsing only succeeds for patterns covered by lipidnames_pattern; molecules outside supported patterns are flagged, not auto-corrected, and require manual RegEx renaming via update_molecule_names
- Full-chemical-name ceramides are a known failure case that must be renamed to shorthand before annotation is inferred correctly

## Evidence

- [abstract] Automatic inference of class/chain from names: "lipidr imports numerical matrices, Skyline exports, and Metabolomics Workbench files directly into R, automatically inferring lipid class and chain information from lipid names"
- [intro] Class, total chain length, and unsaturation as annotation dimensions: "A novel lipid set enrichment analysis (LSEA) is implemented to detect preferential enrichment of certain lipid classes, total chain lengths or unsaturation patterns."
- [intro] Example dataset ST001111 supplying the molecule name strings: "The example shown in this vignette is from `ST001111`"
