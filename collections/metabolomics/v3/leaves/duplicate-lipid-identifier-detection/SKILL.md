---
name: duplicate-lipid-identifier-detection
description: Use when a CSV/TXT lipidomic data matrix is about to be ingested and
  you must guarantee row identifiers (lipid names) are unique before grouping, zero-variance
  checks, or statistics run.
when_to_use_negative:
- The input is not a lipid species matrix keyed by lipid-name rows (e.g. it is already
  a validated feature table or a results/statistics output)
- Only aggregate statistics are needed and the matrix has already passed an upload
  validation step
- The data comes from the Lipid One 2.0 web tool itself, which already performs duplicate
  and label checking at upload time
edam_operation: http://edamontology.org/operation_3757
tools:
- name: Lipid One 2.0
  role: Reference web implementation whose upload step detects duplicate lipid names
    and reports them explicitly before accepting the dataset
  repo: https://lipidone.eu
- name: R
  role: Language of the Lipid One 2.0 back-end scripts implementing dataset upload
    and validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:abe116b57960391ea155575bd38c10dfe36946edd451e21378492259564fb040
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lipid One 2.0
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_3757
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# duplicate-lipid-identifier-detection

## Summary

A pre-upload validation procedure for lipidomic data matrices that scans the lipid-name column for identifiers appearing more than once and rejects the upload with an error report naming each duplicated lipid. Mirrors the upload-time checking behavior of Lipid One 2.0 (https://lipidone.eu) so duplicate-row errors are caught before statistical analysis.

## When to use

Use when a CSV/TXT lipidomic data matrix is about to be ingested and you must guarantee row identifiers (lipid names) are unique before grouping, zero-variance checks, or statistics run.

## When NOT to use

- The input is not a lipid species matrix keyed by lipid-name rows (e.g. it is already a validated feature table or a results/statistics output)
- Only aggregate statistics are needed and the matrix has already passed an upload validation step
- The data comes from the Lipid One 2.0 web tool itself, which already performs duplicate and label checking at upload time

## Inputs

- Lipidomic data matrix file in CSV or TXT format, with lipid names as rows and samples as columns
- Sample-to-group label table (required label fields, per-sample group assignments)

## Outputs

- Structured validation error report (JSON with per-check error lists: duplicates, zero/constant rows, label violations), written to validation_report.json
- List of duplicated lipid names to surface in an upload error message

## How to apply

Load the lipidomic data matrix (CSV/TXT) with pandas, treating lipid names as the row index and samples as columns. Scan the lipid-name index and record every lipid name that occurs more than once, collecting the exact duplicated names rather than just a boolean pass/fail. This check should be combined with companion upload checks from the Lipid One 2.0 ingestion pipeline: verifying that required sample/group label fields are present and well-formed, and flagging lipid rows that are all-zero or constant within a group. Emit all detected issues as a structured error report (JSON with per-check error lists), writing it to validation_report.json. Rationale: Lipid One 2.0 refuses to accept an upload containing duplicate lipid names and instead displays an error message explicitly indicating which lipid names are duplicated, so any reimplementation must preserve identifier-level specificity, not merely reject the file.

## Related tools

- **Lipid One 2.0** (Reference web implementation whose upload step detects duplicate lipid names and reports them explicitly before accepting the dataset) — https://lipidone.eu
- **R** (Language of the Lipid One 2.0 back-end scripts implementing dataset upload and validation)

## Examples

```
import pandas as pd; df = pd.read_csv('lipid_matrix.csv', index_col=0); dup = df.index[df.index.duplicated()].unique(); assert len(dup)==0, f'Duplicate lipids: {list(dup)}'
```

## Evaluation signals

- validation_report.json contains a 'duplicates' section, a zero/constant-rows section, and a label-violations section
- Every entry in the duplicates section names the specific offending lipid name, not just a count of conflicts
- Uploading a matrix with two rows sharing one lipid name fails validation while the same matrix with unique names passes
- All-zero or group-constant lipid rows are flagged per group label, with the sample/group label named in each entry
- No duplicated lipid name reaches downstream statistics (the upload is rejected whenever duplicates exist)

## Limitations

- The Lipid One 2.0 paper section provided does not state the software license for the web tool or whether its source code is publicly hosted; the linked GitHub README describes the older Perl/Prima desktop LipidOne for Microsoft Windows (FreeBSD two-clause license), not the 2.0 web tool
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform rather than a persistent repository accession, complicating reproducible validation tests
- No installability/deployment information (Docker, conda, source install) is given for Lipid One 2.0, so local reimplementation of the validation step may be required

## Evidence

- [abstract] LipidOne 2.0 detects duplicate lipid names and displays an error message indicating which names are duplicated: "LipidOne 2.0 (https://lipidone.eu) is a new web bioinformatic tool for the analysis of lipidomic data."
- [abstract] Upload protocol defines the ingestion step where duplicate checking occurs: "Basic Protocol 2: Uploading a dataset into LipidOne 2.0"
- [readme] README confirms the repository corresponds to the LipidOne desktop software for lipidomic dataset analysis: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
- [intro] Lipid One 2.0 is the lipid-oriented data analysis platform whose back-end is written in R: [evidence span withheld — anchor status: not_found]
