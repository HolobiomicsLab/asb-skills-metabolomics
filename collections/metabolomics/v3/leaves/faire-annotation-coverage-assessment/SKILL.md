---
name: faire-annotation-coverage-assessment
description: Use when auditing a PeakForest compound card or demonstrator compound
  table to compute identifier-coverage fractions per external chemical bank (ChEBI,
  PubChem, HMDB, KEGG) and InChIKey completeness.
when_to_use_negative:
- The input is raw spectral data — PeakForest is designed for metabolite annotation
  report data rather than storage of raw spectral data, so identifier coverage is
  not applicable there
- The compound table has no cross-reference or InChIKey columns — PeakForest requires
  metadata to be manually entered, so coverage cannot be computed from absent fields
- The input is already an aggregated coverage/summary table — recomputing counts would
  double-aggregate and lose the per-compound denominator
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2250
tools:
- name: PeakForest
  role: Source of the compound card whose cross-reference identifiers section (ChEBI,
    PubChem, KEGG, HMDB) is the basis for computing per-resource coverage
  repo: https://github.com/peakforest/peakforest-webapp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-022-01899-3
    title: 'PeakForest: a multi-platform digital infrastructure for interoperable
      metabolite spectral data and metadata management'
  source_content_hash: sha256:ea507c5e3c49aea7962843faecae7074d307046dcd9cce9ecd2e39b0b09665ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/peakforest/peakforest-webapp
  tools:
  - PeakForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2250
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# faire-annotation-coverage-assessment

## Summary

Quantifies per-resource cross-reference identifier coverage (ChEBI, PubChem, HMDB, KEGG, plus InChIKey as a persistent-identifier completeness metric) over a PeakForest compound-card demonstrator table, in line with PeakForest's FAIR-aligned annotation recommendations.

## When to use

Use when auditing a PeakForest compound card or demonstrator compound table to compute identifier-coverage fractions per external chemical bank (ChEBI, PubChem, HMDB, KEGG) and InChIKey completeness.

## When NOT to use

- The input is raw spectral data — PeakForest is designed for metabolite annotation report data rather than storage of raw spectral data, so identifier coverage is not applicable there
- The compound table has no cross-reference or InChIKey columns — PeakForest requires metadata to be manually entered, so coverage cannot be computed from absent fields
- The input is already an aggregated coverage/summary table — recomputing counts would double-aggregate and lose the per-compound denominator

## Inputs

- 96-compound PeakForest demonstrator table (CSV/spreadsheet) with cross-reference identifier columns for ChEBI, PubChem, HMDB, KEGG, LIPID MAPS, MassBank, MoNA, mzCloud
- InChIKey column from the same demonstrator table

## Outputs

- coverage_table.csv with one row per resource: resource name, non-empty count, coverage percentage
- InChIKey row included as persistent-identifier completeness metric

## How to apply

Load the 96-compound demonstrator table into pandas and read the cross-reference identifier columns for ChEBI, PubChem, HMDB, KEGG, LIPID MAPS, MassBank, MoNA and mzCloud, plus the InChIKey column. For each external resource, count compounds carrying a non-empty identifier and compute the coverage fraction with denominator 96 (PeakForest's compound card implements a cross-reference section referencing exactly four selected external banks: ChEBI, PubChem, KEGG and HMDB). Compute InChIKey coverage separately, treating its presence as the persistent-identifier completeness metric aligned with PeakForest's FAIR annotation recommendations. Assemble a summary table with one row per resource (resource name, non-empty count, coverage percentage) plus an InChIKey row, and write it to coverage_table.csv. Validate that every count lies in [0, 96], every percentage within 0–100%, and that the InChIKey row is present in the output file.

## Related tools

- **PeakForest** (Source of the compound card whose cross-reference identifiers section (ChEBI, PubChem, KEGG, HMDB) is the basis for computing per-resource coverage) — https://github.com/peakforest/peakforest-webapp

## Examples

```
df = pd.read_csv('demonstrator.csv'); res = ['ChEBI','PubChem','HMDB','KEGG','LIPID MAPS','MassBank','MoNA','mzCloud','InChIKey']; cov = pd.DataFrame([{'resource': r, 'count': int(df[r].notna().sum())} for r in res]); cov.to_csv('coverage_table.csv', index=False)
```

## Evaluation signals

- Every per-resource count lies within [0, 96] (the demonstrator compound denominator)
- Every coverage percentage lies within 0–100%
- coverage_table.csv contains an InChIKey row in addition to the external-bank rows
- One summary row exists per queried resource (ChEBI, PubChem, HMDB, KEGG, LIPID MAPS, MassBank, MoNA, mzCloud) with non-empty count and percentage columns populated
- ChEBI, PubChem, KEGG and HMDB rows correspond to the four external banks implemented in the PeakForest compound card

## Limitations

- PeakForest does not include data clean-up or extraction tools and requires metadata to be manually entered in the database, so cross-reference fields may be sparsely populated
- A minimum number of compounds and spectra must be imported before local PeakForest databases are fully functional, which constrains the denominator for small instances
- Initial database construction takes time and deployment is iterative, so coverage metrics reflect curation progress rather than absolute annotation quality

## Evidence

- [abstract] FAIR-aligned formats rationale: "Standardized operating procedures and formats were included to ensure data quality and interoperability, in line with international recommendations and FAIR principles."
- [abstract] PeakForest scope (annotation metadata, not raw data): "PeakForest is able to capture and store experimental spectral MS and NMR metadata as well as collect and display signal annotations."
- [methods] Open-source publication of PeakForest: "PeakForest is a free and open-source project under the CECILL-2.16 license, published on GitHub."
- [discussion] Metadata entry limitation: "PeakForest does not include data clean-up or extraction tools per se and requires metadata to be manually entered in the database"
- [readme] WebApp provides compound library services: "chemical compound library manager"
