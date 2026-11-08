---
name: sample-design-metadata-annotation
description: Use when you have a wide-format metabolomics feature table (samples as
  columns, features as rows) and need to attach sample-class annotations (e.g. blank
  vs treatment group) from a design file before running QC or statistics.
when_to_use_negative:
- Data is already in a long/melted (tidy) format where each row is a single feature-by-sample
  measurement and no design-file alignment is needed.
- Samples lack any companion design/annotation file, so blank vs treatment-group partitioning
  cannot be performed.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SECIMTools
  role: Provides the bundled example wide-format feature table and design file, and
    consumes the annotation to partition samples for QC steps such as Blank Feature
    Filtering.
  repo: https://github.com/SECIMTools
- name: Galaxy
  role: Wraps SECIMTools so the annotated tab/tabular output can be piped between
    tools in a workflow.
- name: Python
  role: Runtime language used to load the table and design file with pandas and perform
    the annotation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:e71414dee1431e694b20efbbf0e96000e81b97c315d2cd8f3cb4a41747a1f550
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Python
  - SECIMTools
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sample-design-metadata-annotation

## Summary

Annotates a wide-format feature intensity table with a companion design file that partitions samples into blank samples versus experimental/treatment-group samples, enabling per-sample grouping and blank-based QC such as Blank Feature Filtering in SECIMTools.

## When to use

Use when you have a wide-format metabolomics feature table (samples as columns, features as rows) and need to attach sample-class annotations (e.g. blank vs treatment group) from a design file before running QC or statistics.

## When NOT to use

- Data is already in a long/melted (tidy) format where each row is a single feature-by-sample measurement and no design-file alignment is needed.
- Samples lack any companion design/annotation file, so blank vs treatment-group partitioning cannot be performed.

## Inputs

- Wide-format feature intensity table (rows = features with unique identifiers, columns = samples), e.g. from SECIMTools bundled test data
- Companion design file with per-sample annotations (treatment groups, blank flags, run order) used by SECIMTools

## Outputs

- Annotated wide-format feature table with sample-group annotations and, optionally, per-feature QC flag columns (e.g. BFF limit-of-detection summary and 0/1 flag) in tab/tabular tsv format compatible with Galaxy workflows

## How to apply

Load the SECIMTools bundled example wide-format feature table (unique feature identifiers as rows, sample columns as intensities) together with its companion design file using pandas. The design file carries sample annotations (e.g. treatment groups, blanks, run order) that SECIMTools uses to partition samples; merge or align these annotations to the feature table by sample column name so downstream tools can act on groups. For Blank Feature Filtering (BFF), use the design file's blank-sample flags to select blank columns, compute a per-feature limit of detection from those blank intensities, and write a 0/1 BFF flag per feature in the output. Output a Galaxy-compatible tab/tabular (tsv) table, retaining original feature identifiers plus added annotation or flag columns, which is the standard format SECIMTools uses to let arbitrary tools be linked into workflows.

## Related tools

- **SECIMTools** (Provides the bundled example wide-format feature table and design file, and consumes the annotation to partition samples for QC steps such as Blank Feature Filtering.) — https://github.com/SECIMTools
- **Galaxy** (Wraps SECIMTools so the annotated tab/tabular output can be piped between tools in a workflow.)
- **Python** (Runtime language used to load the table and design file with pandas and perform the annotation.)

## Examples

```
python -c "import pandas as pd; feats = pd.read_csv('features.csv'); design = pd.read_csv('design.tsv', sep='\t'); feats.merge(design[['sample','treatment','blank']], left_on='sample', right_on='sample')"
```

## Evaluation signals

- Output CSV/TSV exists and is in wide format with the same feature rows and identifiers as the input table.
- Every sample column in the output carries a non-missing annotation matching the design file (blank vs treatment group).
- Where BFF is applied, the output contains a per-feature limit-of-detection summary field and a 0/1 flag column.
- Features at or below the blank-derived limit of detection are flagged 1, and only above-threshold features are retained in the filtered table.
- File parses as tab/tabular (tsv) and is importable into Galaxy as the SECIMTools pipeline expects.

## Limitations

- Annotation depends entirely on a correctly formatted companion design file; missing or mismatched sample names will silently leave groups unassigned.
- No version number for SECIMTools or Galaxy is given in the article, and the GitHub reference is not resolved in the text, so tool-version provenance must be tracked separately.
- The article gives no threshold default for the limit-of-detection calculation beyond 'user supplied' BFF parameters, so the blank cutoff must be chosen by the analyst.

## Evidence

- [other] BFF partitioning input: [evidence span withheld — anchor status: not_found]
- [methods] design-file sample grouping: "The files available in both public repositories can be imported into Galaxy and used in SECIMTools."
- [abstract] standard data formats enable arbitrary tool linkages: "Standard data formats and a set of utilities allow arbitrary linkages between tools to encourage novel workflow designs."
- [readme] python package availability: "SECIMTools are available as a secimtools pypi package. Project has also been packaged for bioconda and Galaxy Genomics Framework."
