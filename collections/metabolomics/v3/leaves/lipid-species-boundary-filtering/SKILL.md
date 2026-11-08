---
name: lipid-species-boundary-filtering
description: Use when preprocessing vendor lipidomics output (e.g., LipidSearch or
  LIQUID tables) whose lipid species names must be validated against class-specific
  carbon-atom and double-bond limits before downstream normalization or statistical
  analysis.
when_to_use_negative:
- Input data comes directly from the Metabolomics Workbench rather than LipidSearch/LIQUID
  vendor outputs whose species names need boundary validation.
- The lipid matrix is already a validated feature table — re-running boundary filtering
  could remove legitimate species.
- The analysis goal is differential abundance testing or batch effect correction,
  which presuppose that boundary filtering has already been applied.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: ADViSELipidomics
  role: Shiny app implementing the parsing of lipid species (LIPID MAPS classification)
    and boundary-based filtering of the lipid list
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor software whose output files supply the lipid species identifiers that
    are parsed and boundary-filtered
- name: LIPID MAPS
  role: Classification system used to parse lipid class, total carbon atoms, and double-bond
    count from species identifiers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:4ba5c88b36540006be00ca45c7b7418690b3d376edc5e5e01ced20d02795a05a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LIPID MAPS
  - LipidSearch
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# lipid-species-boundary-filtering

## Summary

A preprocessing step in ADViSELipidomics that parses lipid species identifiers using LIPID MAPS classification (class, total carbons, double-bond count) and removes species violating double-bond or carbon-atom boundaries defined in an internal reference file, duplicating entries, or having odd-chain structures.

## When to use

Use when preprocessing vendor lipidomics output (e.g., LipidSearch or LIQUID tables) whose lipid species names must be validated against class-specific carbon-atom and double-bond limits before downstream normalization or statistical analysis.

## When NOT to use

- Input data comes directly from the Metabolomics Workbench rather than LipidSearch/LIQUID vendor outputs whose species names need boundary validation.
- The lipid matrix is already a validated feature table — re-running boundary filtering could remove legitimate species.
- The analysis goal is differential abundance testing or batch effect correction, which presuppose that boundary filtering has already been applied.

## Inputs

- Vendor lipid identification/quantification output files (LipidSearch or LIQUID), e.g., from Case Study 1
- Internal reference file defining per-class boundaries on total carbon atoms and double-bond counts

## Outputs

- Filtered lipid list with one entry per retained lipid species (violating species removed, duplicates dropped, odd-chain species flagged)

## How to apply

Parse each lipid species identifier using the LIPID MAPS classification to extract three attributes: lipid class, total carbon-atom count, and double-bond count. Load the internal reference file that defines, per lipid class, the admissible boundaries on carbon atoms and double bonds. Remove any species whose parsed values violate these limits, drop duplicate entries detected during parsing, and flag odd-chain species (odd total carbon number) rather than silently deleting them. Retain only species passing all checks so the downstream normalization and SummarizedExperiment assembly operates on a validated lipid list.

## Related tools

- **ADViSELipidomics** (Shiny app implementing the parsing of lipid species (LIPID MAPS classification) and boundary-based filtering of the lipid list) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor software whose output files supply the lipid species identifiers that are parsed and boundary-filtered)
- **LIPID MAPS** (Classification system used to parse lipid class, total carbon atoms, and double-bond count from species identifiers)

## Examples

```
library(ADViSELipidomics); run_ADViSELipidomics()
```

## Evaluation signals

- No retained lipid species violates the double-bond or carbon-atom limits for its class as defined in the internal reference file.
- No duplicate lipid species entries remain in the filtered list.
- Odd-chain species are flagged (not silently dropped) so their treatment is auditable.
- The filtered lipid count matches one row per retained species in the downstream SummarizedExperiment concentration matrix.
- Every retained species identifier yields a parseable lipid class, carbon-atom count, and double-bond count under LIPID MAPS classification.

## Limitations

- Species identifiers that do not conform to LIPID MAPS naming conventions may fail to parse into class, carbon count, and double-bond count.
- Boundary definitions are only as good as the internal reference file; overly restrictive limits can discard real species.
- The workflow is designed for LipidSearch and LIQUID outputs and Metabolomics Workbench data; other vendor formats may not be supported.

## Evidence

- [abstract] Boundary filtering uses LIPID MAPS parsing plus reference-file limits: "extracts information by parsing lipid species (using LIPID MAPS classification)"
- [supplementary] Filtering is part of reconstructing the preprocessing module: [evidence span withheld — anchor status: not_found]
- [supplementary] Filtering happens before normalization and SummarizedExperiment assembly: [evidence span withheld — anchor status: not_found]
- [readme] ADViSELipidomics handles LipidSearch and LIQUID vendor outputs: "It copes with the outputs from LipidSearch and LIQUID for lipid identification and quantification"
