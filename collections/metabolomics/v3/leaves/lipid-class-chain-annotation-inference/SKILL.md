---
name: lipid-class-chain-annotation-inference
description: Use when importing lipidomics matrices, Skyline exports, or Metabolomics
  Workbench studies into lipidr and you need lipid class/chain annotations parsed
  from molecule names, or need to detect unparseable names.
when_to_use_negative:
- Input is already a feature table with validated, lipidr-parseable shorthand lipid
  names and no annotation step is needed
- The dataset's molecule names use nomenclature lipidr does not support and cannot
  be corrected by renaming (e.g. free-form chemical descriptors without a pattern-fix)
- The goal is annotation of MS/MS spectra rather than annotation from molecule name
  strings
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lipidr
  role: R/Bioconductor package performing the import and automatic inference of lipid
    class and chain information from lipid names, plus non_parsed_molecules() detection
    of unparseable names
  repo: github.com/ahmohamed/lipidr
- name: Metabolomics Workbench API
  role: Source of public lipidomics studies (e.g. ST001111) whose molecule names are
    parsed on import
- name: R/Bioconductor
  role: Runtime environment in which the import, name parsing, and regex-based renaming
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:12e294e86019ddeb7bbf809113dc72b81dcc089f97e47db8af82804a2caa0713
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Metabolomics Workbench API
  - R/Bioconductor
  - lipidr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lipid-class-chain-annotation-inference

## Summary

Infers lipid class and fatty-acyl chain information directly from lipid molecule names during import of lipidomics datasets into lipidr, and identifies molecules whose names do not follow supported nomenclature so they can be renamed before analysis.

## When to use

Use when importing lipidomics matrices, Skyline exports, or Metabolomics Workbench studies into lipidr and you need lipid class/chain annotations parsed from molecule names, or need to detect unparseable names.

## When NOT to use

- Input is already a feature table with validated, lipidr-parseable shorthand lipid names and no annotation step is needed
- The dataset's molecule names use nomenclature lipidr does not support and cannot be corrected by renaming (e.g. free-form chemical descriptors without a pattern-fix)
- The goal is annotation of MS/MS spectra rather than annotation from molecule name strings

## Inputs

- Numerical matrix CSV (lipids as rows, samples as columns, lipid names in first column)
- Skyline-exported results CSV
- Metabolomics Workbench study ID (e.g. ST001111) fetched via the MW API
- Sample annotation/clinical CSV with matching sample names

## Outputs

- LipidomicsExperiment (SummarizedExperiment extension) with inferred lipid class and chain annotations
- Vector/table of non-parsed molecule names (e.g. written to non_parsed_molecules.csv)
- Corrected molecule names compatible with lipidr nomenclature parsing

## How to apply

Import the dataset with lipidr (as_lipidomics_experiment, read_skyline, or fetch_mw_study), which automatically infers lipid class and chain information from lipid names. Inspect the resulting LipidomicsExperiment molecule names to confirm parsing. Run non_parsed_molecules() on the object to enumerate names that fail lipidr's supported nomenclature patterns; for Metabolomics Workbench study ST001111 these are ceramides written as full chemical names. Correct such names with a regular expression that replaces the leading part with 'Cer (' so that all molecules parse and downstream quality control, normalization, and differential analysis operate on complete annotations.

## Related tools

- **lipidr** (R/Bioconductor package performing the import and automatic inference of lipid class and chain information from lipid names, plus non_parsed_molecules() detection of unparseable names) — github.com/ahmohamed/lipidr
- **Metabolomics Workbench API** (Source of public lipidomics studies (e.g. ST001111) whose molecule names are parsed on import)
- **R/Bioconductor** (Runtime environment in which the import, name parsing, and regex-based renaming are executed)

## Examples

```
d <- fetch_mw_study("ST001111"); non_parsed_molecules(d)
```

## Evaluation signals

- non_parsed_molecules() on the imported object returns an empty set (all molecule names parsed)
- Inferred chain/class annotations are present for every molecule in the LipidomicsExperiment after import
- For ST001111, the regex renaming converts the full-chemical-name ceramide entries to 'Cer (...)' form and molecule count is unchanged
- Imported sample names in the LipidomicsExperiment exactly match those in the sample annotation file

## Limitations

- Molecule names not following lipidr's supported nomenclature patterns are left unannotated; ST001111's full-chemical-name ceramides are a known example requiring a regex fix
- Automatic annotation is name-based, so incorrect or ambiguous lipid names propagate as incorrect class/chain inference

## Evidence

- [readme] LipidomicsExperiment object used to carry annotations: [evidence span withheld — anchor status: external_doc]
