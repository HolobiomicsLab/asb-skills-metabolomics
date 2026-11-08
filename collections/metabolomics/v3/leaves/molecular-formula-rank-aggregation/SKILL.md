---
name: molecular-formula-rank-aggregation
description: Use when IDSL.IPA has generated individual per-sample peaklists and an
  aligned peak table from untargeted LC/HRMS (mzXML/mzML/netCDF) and you need to consolidate
  formula annotations across samples into a summary table ranked by frequency and
  median rank.
when_to_use_negative:
- Input is a single-sample peaklist with no aligned peak table — aggregation across
  samples requires the IPA peak_alignment output
- Only MS2 spectra are available without MS1 isotopic profiles — this skill relies
  on MS1 isotopic-profile matching, not MS/MS library search
- You need exhaustive elemental composition enumeration for unknowns (use IDSL.UFAx
  instead)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.UFA
  role: Aggregates per-sample formula annotations into the aligned annotated peak
    table using detection frequency and median rank; generates the ranked hit list
    per peak
  repo: https://github.com/idslme/IDSL.UFA
- name: IDSL.IPA
  role: Produces the individual peaklists and aligned peak table that are the required
    input for the aggregation step
  repo: https://github.com/idslme/IDSL.IPA
- name: IDSL.UFAx
  role: Optional companion for exhaustive chemical-space enumeration when a prioritized
    IPDB formula source is insufficient
  repo: https://github.com/idslme/IDSL.UFAx
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:5cb07ccb4fb1d9c3e52c0a4f081a6bfffcd0d02691548269f2c947b28b5c8f20
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idslme/IDSL.UFA
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - IDSL.UFAx
  - R
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-formula-rank-aggregation

## Summary

Aggregate per-sample molecular formula annotation lists from IDSL.UFA into an aligned annotated peak table using detection frequency and median rank statistics, then rank formula hits per peak (e.g. median rank ≤ 5, top rank ≤ 30) across individual samples in population-scale LC/HRMS studies.

## When to use

Use when IDSL.IPA has generated individual per-sample peaklists and an aligned peak table from untargeted LC/HRMS (mzXML/mzML/netCDF) and you need to consolidate formula annotations across samples into a summary table ranked by frequency and median rank.

## When NOT to use

- Input is a single-sample peaklist with no aligned peak table — aggregation across samples requires the IPA peak_alignment output
- Only MS2 spectra are available without MS1 isotopic profiles — this skill relies on MS1 isotopic-profile matching, not MS/MS library search
- You need exhaustive elemental composition enumeration for unknowns (use IDSL.UFAx instead)

## Inputs

- Individual per-file formula annotation peaklists from IDSL.UFA (one per HRMS data file)
- Aligned peak table from IDSL.IPA peak_alignment directory
- UFA parameter spreadsheet (UFA_parameters.xlsx) with PARAM0004–PARAM0014 configured
- Pre-calculated IPDB (.Rdata) isotopic-profile database (e.g. IDSL.ExposomeDB or RefMetDB)
- MS1-level HRMS data files (mzXML, mzML, netCDF)

## Outputs

- Aligned annotated peak table (CSV) with per-peak formula hits ranked by detection frequency and median rank
- Per-peak top 5–20 molecular formula hits with median rank and frequency statistics
- Formula-class grouping of aligned formulas (optional downstream step)

## How to apply

Run IDSL.UFA with PARAM0005 and PARAM0006 set to YES, supplying the peaklists and peak_alignment directories produced by the IDSL.IPA workflow plus an isotopic-profile database (IPDB, e.g. from IDSL.ExposomeDB or RefMetDB). For each aligned peak, IDSL.UFA counts in how many samples each candidate molecular formula was annotated (detection frequency) and computes the median rank of that formula across those samples, then re-sorts hits by frequency/median rank. Filter the resulting aligned annotated peak table to peaks with at least one formula having median rank ≤ 5, and cap unique formulas at top rank ≤ 30 and number of hits ≤ 30. Export the table as CSV retaining roughly the top 5–20 formula hits per peak for downstream interpretation.

## Related tools

- **IDSL.UFA** (Aggregates per-sample formula annotations into the aligned annotated peak table using detection frequency and median rank; generates the ranked hit list per peak) — https://github.com/idslme/IDSL.UFA
- **IDSL.IPA** (Produces the individual peaklists and aligned peak table that are the required input for the aggregation step) — https://github.com/idslme/IDSL.IPA
- **IDSL.UFAx** (Optional companion for exhaustive chemical-space enumeration when a prioritized IPDB formula source is insufficient) — https://github.com/idslme/IDSL.UFAx

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Every reported peak in the aligned annotated peak table should have at least one molecular formula with median rank ≤ 5 across individual samples
- For ST002044 MS/MS-confirmed hits, ~78.75% should show median rank ≤ 2 — a benchmark for true-positive agreement
- Number of unique formulas should stay within top rank ≤ 30 and number of hits ≤ 30 caps (article reports 199,837 unique formulas under this filter)
- Each peak's retained hits should carry both a detection-frequency count and a median-rank value; missing either indicates the aggregation step was skipped
- PARAM0005 and PARAM0006 must be set to YES in UFA_parameters.xlsx or no aligned-annotation summary is produced

## Limitations

- Aggregation depends on IDSL.IPA's peak alignment quality; misaligned peaks across samples yield misleading frequency/median-rank statistics
- Only MS1 data is required and used — formula assignments are not structurally resolved without MS/MS confirmation
- Requires a pre-built IPDB formula source; compounds absent from that source cannot be annotated without switching to an enumeration approach (IDSL.UFAx)
- Thresholds (median rank ≤ 5, top rank ≤ 30, hits ≤ 30) are study-tuned values from the pregnancy-exposome application and may need adjustment for other cohort designs

## Evidence

- [results] For each peak in the aligned table, IDSL.UFA aggregates formula lists across samples and computes detection frequency and median rank per formula, then generates a new sort order using the formula frequency/median rank.: [evidence span withheld — anchor status: not_found]
- [readme] PARAM0005/PARAM0006 gating for aligned annotation: [evidence span withheld — anchor status: not_found]
- [other] 78.75% MS/MS-confirmed hits with median rank ≤ 2 in ST002044: [evidence span withheld — anchor status: not_found]
