---
name: lipidomics-data-import-from-skyline-exports
description: Use when targeted lipidomics measurements have been exported from Skyline
  as a CSV file and need to be loaded into R for lipidr-based quality control, normalization,
  summarization, or differential analysis.
when_to_use_negative:
- Input is already a numerical intensity matrix (rows = lipids, columns = samples)
  — use lipidr's as_lipidomics_experiment instead of read_skyline
- Data is to be fetched directly from a public study — use lipidr's Metabolomics Workbench
  API integration (e.g. study ST001111) rather than a local Skyline export
- You only have non-Skyline vendor raw files that have not been exported from Skyline
  — no CSV export exists yet for read_skyline to parse
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: lipidr
  role: Provides read_skyline and add_sample_annotation to parse Skyline CSV exports
    into an annotated LipidomicsExperiment
  repo: https://github.com/ahmohamed/lipidr
- name: Skyline
  role: Source of the CSV results export that serves as the raw input for read_skyline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.jproteome.0c00082
    title: 'lipidr: A Software Tool for Data Mining and Analysis of Lipidomics Datasets'
  source_content_hash: sha256:76e64015300876dcfb1fa5e3b672c5724befe59b92282d7d6586d1be38f1922e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ahmohamed/lipidr
  tools:
  - Skyline
  - lipidr
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lipidomics-data-import-from-skyline-exports

## Summary

Import targeted lipidomics results exported from Skyline as CSV into R using lipidr's read_skyline, which parses transition-level measurements and automatically infers lipid class and chain information from lipid names. This is the entry point of the lipidr workflow, producing a LipidomicsExperiment that supports QC, normalization, and differential analysis.

## When to use

Use when targeted lipidomics measurements have been exported from Skyline as a CSV file and need to be loaded into R for lipidr-based quality control, normalization, summarization, or differential analysis.

## When NOT to use

- Input is already a numerical intensity matrix (rows = lipids, columns = samples) — use lipidr's as_lipidomics_experiment instead of read_skyline
- Data is to be fetched directly from a public study — use lipidr's Metabolomics Workbench API integration (e.g. study ST001111) rather than a local Skyline export
- You only have non-Skyline vendor raw files that have not been exported from Skyline — no CSV export exists yet for read_skyline to parse

## Inputs

- Skyline results export as CSV (transition-level measurements, e.g. inst/extdata/A1_data.csv)
- Sample annotation CSV with sample names in the first column (e.g. inst/extdata/clin.csv)

## Outputs

- LipidomicsExperiment object (extends SummarizedExperiment) with inferred lipid class and chain information and attached sample annotations

## How to apply

Call lipidr's read_skyline on the Skyline-exported results CSV (e.g. the A1_data.csv-style export with transition-level rows), which returns a LipidomicsExperiment — an object extending SummarizedExperiment to ease integration with other Bioconductor packages. read_skyline automatically infers lipid class and chain information from the lipid names and allows multiple Skyline methods to be analyzed together in one object. Immediately attach sample annotations with add_sample_annotation using a CSV where sample names are in the first column; the sample names must be identical in the export and the annotation file (as in the lipidr extdata clin.csv paired with the Skyline export). Once imported, downstream lipidr steps (molecule summarization, PQN normalization, de_analysis via limma) operate on this annotated LipidomicsExperiment.

## Related tools

- **lipidr** (Provides read_skyline and add_sample_annotation to parse Skyline CSV exports into an annotated LipidomicsExperiment) — https://github.com/ahmohamed/lipidr
- **Skyline** (Source of the CSV results export that serves as the raw input for read_skyline)

## Examples

```
d <- read_skyline("Skyline_export.csv"); d <- add_sample_annotation(d, "clin.csv")
```

## Evaluation signals

- read_skyline returns a LipidomicsExperiment (SummarizedExperiment derivative) rather than a plain data.frame
- Lipid class and chain annotations are populated automatically from lipid names after import
- Sample names in the imported object exactly match the sample names in the annotation CSV — add_sample_annotation will fail or mis-annotate if they differ
- Multiple Skyline methods can be analyzed together, so the object retains measurements across methods in a single container
- Imported object supports downstream lipidr steps: summarization to molecule level, PQN normalization, and de_analysis

## Limitations

- Sample names in the Skyline export and the sample-annotation CSV must be identical; mismatches break annotation attachment
- Import expects a Skyline-style CSV export; raw vendor files or non-Skyline formats are not handled by read_skyline
- The paired annotation file must have sample names in the first column (specific format required by lipidr)

## Evidence

- [readme] Sample-name matching requirement for the annotation file: [evidence span withheld — anchor status: external_doc]
- [readme] LipidomicsExperiment object representation: [evidence span withheld — anchor status: external_doc]
