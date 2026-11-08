---
name: lipid-class-annotation-from-names
description: Use when lipid feature identifiers are name strings (e.g. from a Skyline
  export or Metabolomics Workbench study like ST001111) and you need class/chain/unsaturation
  annotations, including renaming full-chemical-name ceramides to shorthand.
when_to_use_negative:
- The dataset already carries class/chain annotation columns, and you only need analysis
  or plotting rather than name parsing
- The task is the downstream statistics itself (mva OPLS-DA, LSEA, differential analysis)
  rather than annotating features
- Feature identifiers are not lipid names (e.g. mzRT features) that cannot match any
  of lipidr's supported naming patterns
tools:
- name: lipidr
  role: R/Bioconductor package whose lipidnames_pattern rules parse molecule names
    into class/chain/unsaturation, whose update_molecule_names renames full-chemical-name
    ceramides to shorthand, and whose import functions automatically infer lipid class
    and chain information
  repo: github.com/ahmohamed/lipidr
- name: R/Bioconductor
  role: Environment in which lipidr runs and in which the annotation mapping table
    is assembled and written to CSV
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:422f146092adcd78b46cedefce675dcb03fcae8814ea886c2a5e6c2d38f03517
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - R/Bioconductor
  - lipidr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lipid-class-annotation-from-names

## Summary

Infers lipid class, total chain length, and unsaturation (double-bond) annotations directly from lipid molecule name strings using lipidr's pattern-matching rules, so downstream analyses (LSEA, loadings plots) can group features by lipid class.

## When to use

Use when lipid feature identifiers are name strings (e.g. from a Skyline export or Metabolomics Workbench study like ST001111) and you need class/chain/unsaturation annotations, including renaming full-chemical-name ceramides to shorthand.

## When NOT to use

- The dataset already carries class/chain annotation columns, and you only need analysis or plotting rather than name parsing
- The task is the downstream statistics itself (mva OPLS-DA, LSEA, differential analysis) rather than annotating features
- Feature identifiers are not lipid names (e.g. mzRT features) that cannot match any of lipidr's supported naming patterns

## Inputs

- CSV/text list or table of lipid molecule name strings (e.g. from a Skyline export or Metabolomics Workbench study such as ST001111)
- Numerical matrix + sample annotation CSV, or a Skyline results CSV, loaded as a LipidomicsExperiment
- Optional RegEx mapping of full-chemical-name ceramides to shorthand notation for update_molecule_names

## Outputs

- Molecule-to-annotation mapping table (e.g. lipid_annotations.csv) with lipid class, total chain length, and unsaturation per molecule
- List/flag of molecule names that failed to match lipidnames_pattern rules
- A LipidomicsExperiment carrying automatic lipid class and chain annotations for downstream plotting/LSEA

## How to apply

Load the molecule name strings and parse each against lipidr's supported naming patterns (lipidnames_pattern regex rules) to extract the lipid class, total chain length, and unsaturation information; names that do not match a supported pattern are flagged. For unparseable molecules — notably full-chemical-name ceramides — rename them to their shorthand equivalents with update_molecule_names using RegEx before re-parsing. Work within lipidr's LipidomicsExperiment container (created via as_lipidomics_experiment or read_skyline and annotated with add_sample_annotation) so the inferred annotations stay attached to the features. Assemble the parsed class/chain/unsaturation fields into a molecule-to-annotation mapping table (e.g. lipid_annotations.csv); these annotations are what lipidr uses for class-level aggregation in LSEA and for coloring loadings plots.

## Related tools

- **lipidr** (R/Bioconductor package whose lipidnames_pattern rules parse molecule names into class/chain/unsaturation, whose update_molecule_names renames full-chemical-name ceramides to shorthand, and whose import functions automatically infer lipid class and chain information) — github.com/ahmohamed/lipidr
- **R/Bioconductor** (Environment in which lipidr runs and in which the annotation mapping table is assembled and written to CSV)

## Examples

```
d <- as_lipidomics_experiment(read.csv("data_matrix.csv")); d <- add_sample_annotation(d, "data_clin.csv")
```

## Evaluation signals

- Every molecule row in the output mapping table carries a non-empty class annotation and chain annotation
- Full-chemical-name ceramide entries have been converted to shorthand notation via update_molecule_names and now parse successfully
- Names that fail to match a supported lipidnames_pattern rule are explicitly flagged rather than silently assigned empty annotations
- Inferred classes agree with lipidr's automatic inference on import (e.g. re-importing the data and checking consistent class assignment)
- Annotations propagate correctly to downstream plots (e.g. loadings colored by lipid class in plot_mva_loadings)

## Limitations

- Only molecule names matching lipidr's supported patterns can be annotated; non-matching names are flagged and remain unannotated
- Full-chemical-name ceramides cannot be parsed as-is and require a user-supplied RegEx rename (update_molecule_names) to shorthand before annotation
- Annotation quality depends on consistent lipid naming conventions in the input files; sample names must be identical across matrix and annotation files

## Evidence

- [readme] lipidr represents lipidomics datasets as a LipidomicsExperiment, which extends SummarizedExperiment, to facilitate integration with other Bioconductor packages.: [evidence span withheld — anchor status: not_found]
