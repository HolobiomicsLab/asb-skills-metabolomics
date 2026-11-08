---
name: bioassay-data-normalization
description: Use when raw bioassay results (e.g., IC50, percent inhibition, growth
  scores) from an extract library arrive in heterogeneous per-assay formats/scales
  and must be assembled into a samples × assays profile matrix before NP Analyst Activity/Cluster
  Score computation.
when_to_use_negative:
- The bioassay columns are already normalized to a common scale and assembled into
  a samples × assays profile matrix
- The task concerns processing the MS side (basketing, replicate comparison, GNPS/MZmine
  2 import) rather than the biological data
- Interpreting NP Analyst outputs (scatter plot, network, community views) — normalization
  is an upstream input-preparation step
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: NP Analyst
  role: Consumes the normalized bioassay profile matrix to compute Activity Scores
    and Cluster Scores; normalization guidelines are documented on its documentation
    site
  repo: https://liningtonlab.github.io/npanalyst_documentation/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:48885b8a8e759794d4a4b6cad73d3d0f92b301c537cdb62442b89c2d035a6433
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine 2
  - NP Analyst
  - Tableau
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bioassay-data-normalization

## Summary

Assembles individual bioassay results for the same extract library into a biological profile matrix whose bioassay columns are normalized onto the same scale, a prerequisite for NP Analyst to compute meaningful Activity Scores and Cluster Scores from the integrated feature–bioassay data.

## When to use

Use when raw bioassay results (e.g., IC50, percent inhibition, growth scores) from an extract library arrive in heterogeneous per-assay formats/scales and must be assembled into a samples × assays profile matrix before NP Analyst Activity/Cluster Score computation.

## When NOT to use

- The bioassay columns are already normalized to a common scale and assembled into a samples × assays profile matrix
- The task concerns processing the MS side (basketing, replicate comparison, GNPS/MZmine 2 import) rather than the biological data
- Interpreting NP Analyst outputs (scatter plot, network, community views) — normalization is an upstream input-preparation step

## Inputs

- Individual bioassay result files for the same extract library (e.g., CSV of extract × assay values such as inhibition %, MIC, or zone sizes)
- Extract/prefraction sample identifiers shared with the MS feature table

## Outputs

- Normalized bioassay CSV (samples × bioassay columns, all columns on the same scale) suitable as NP Analyst input alongside the basketed feature table

## How to apply

Collect the individual bioassay results measured on the same extract library and assemble them into a matrix of samples (rows) × bioassay columns (columns). Normalize each column so that every bioassay uses the same scale — for example, rescale each assay to a common 0–1 range relative to its maximum response — because NP Analyst's Activity Score sums squared bioactivity values across assay columns and is only meaningful if the columns are comparable. The paper accepts 'bioassay data of almost any type', so the normalization scheme must be chosen per assay type (e.g., percent-of-max for dose-response or inhibition data), and detailed normalization instructions are maintained in the NP Analyst website documentation rather than the paper itself. Ensure that sample/extract identifiers in the normalized matrix match the basketed MS feature table (CSV of features × samples with retention time and m/z columns) so the two can be joined. Validate by spot-checking that no column dominates the Activity Score purely by scale and that active extracts retain high values after rescaling.

## Related tools

- **NP Analyst** (Consumes the normalized bioassay profile matrix to compute Activity Scores and Cluster Scores; normalization guidelines are documented on its documentation site) — https://liningtonlab.github.io/npanalyst_documentation/

## Examples

```
import pandas as pd; bio = pd.read_csv('bioassay_results.csv', index_col=0); bio_norm = bio / bio.max(axis=0); bio_norm.to_csv('bioassay_normalized.csv')
```

## Evaluation signals

- All bioassay columns fall on a common scale (e.g., each column's maximum ≈ 1 or comparable ranges), so no single assay dominates the summed Activity Score by magnitude alone
- Sample/extract identifiers in the normalized matrix align 1:1 with the columns of the basketed MS feature table
- Recomputing an Activity Score as the sum of squared normalized bioactivity values across samples containing a feature reproduces the expected ranking of known bioactive features
- No NaNs or mismatched rows remain after joining the normalized bioassay CSV with the feature table

## Limitations

- The paper itself provides only a general statement about normalization; concrete per-assay normalization instructions are deferred to the npanalyst_documentation website
- Because bioassay data of almost any type is accepted, the choice of normalization scheme (percent-of-max vs z-score, handling of qualitative assays) is a user decision that can change Activity Score rankings
- Normalization does not address assay sensitivity limits — an insensitive assay can still inflate network complexity without adding biologically informative MS features

## Evidence

- [discussion] Assemble individual bioassay results for the same extract library into biological profiles with normalized columns on the same scale: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
- [discussion] Normalization instructions are in the website documentation: "Detailed instructions for data normalization are included in the Web site documentation (https://liningtonlab.github.io/npanalyst_documentation/)."
- [abstract] NP Analyst accepts bioassay data of almost any type: "accepts bioassay data of almost any type, and is compatible with mass spectrometry data from major instrument manufacturers via the mzML format"
