---
name: annotation-rate-statistics-computation
description: Use when a GNPS FBMN job's feature counts, ion identity network groups,
  singleton/edge counts, and library- vs in-silico-annotation rates must be reproduced,
  cleaned, and written to a summary table for validation.
when_to_use_negative:
- When the goal is extract novelty prioritization rather than network statistics —
  use the full Inventa Priority rank (FC, LC, CC, SC components) instead
- When the input is raw LC-MS/MS data not yet processed by MZmine2/MZmine3 into a
  feature quantitative table and GNPS job
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GNPS
  role: Runs the Feature Based Molecular Networking job that produces the node, edge,
    IIN and spectral-annotation results being counted
- name: MZmine2
  role: Produces the feature quantitative table ('Peak area', 'row m/z', 'row retention
    time') used as FBMN input
  repo: https://github.com/luigiquiros/inventa
- name: MZmine3
  role: Alternative source of the quantitative table; the data_process_origin parameter
    must be set to 'MZMine3' if Ion Identity is used
  repo: https://github.com/luigiquiros/inventa
- name: SIRIUS
  role: Provides in-silico annotations (ISDB, CANOPUS) filtered at min_ZodiacScore
    = 0.9 and min_ConfidenceScore = 0.0 to complement library annotations
- name: timaR
  role: Supplies taxonomically reponderated in-silico annotation results used alongside
    GNPS spectral matches
  repo: https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- name: Inventa
  role: Implements the annotation cleanup step and computes annotation-related statistics
    over the feature table
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:a152c379ae81360b624004427cd73b6c103cefcf4ca4871c30a68cfdde3884b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  - SIRIUS
  - timaR
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# annotation-rate-statistics-computation

## Summary

Reproduce and report feature/node/edge/annotation statistics for a GNPS Feature Based Molecular Network (e.g. the Celastraceae set: 16,139 nodes, 5,922 singletons, 22,656 edges, 14,554 IIN groups) and compute library and in-silico annotation rates after filtering (ppm_error = 5, shared_peaks = 10, cosine = 0.7; ca. 68% combined annotation rate).

## When to use

Use when a GNPS FBMN job's feature counts, ion identity network groups, singleton/edge counts, and library- vs in-silico-annotation rates must be reproduced, cleaned, and written to a summary table for validation.

## When NOT to use

- When the goal is extract novelty prioritization rather than network statistics — use the full Inventa Priority rank (FC, LC, CC, SC components) instead
- When the input is raw LC-MS/MS data not yet processed by MZmine2/MZmine3 into a feature quantitative table and GNPS job

## Inputs

- GNPS FBMN job input files (GNPS-format metadata table, MZmine quantitative table with 'Peak area', 'row m/z', 'row retention time' columns, GNPS job ID)
- GNPS FBMN output files (node attributes table, edge/IIN results, spectral match results)
- SIRIUS/ISDB in-silico annotation results (optional)
- timaR reponderated annotation output (optional)

## Outputs

- TSV summary file with feature count, IIN group count, node count, singleton count, edge count, annotated nodes before and after cleaning, and overall annotation rate
- Cleaned annotation table (thresholded at ppm_error = 5, shared_peaks = 10, cosine = 0.7)

## How to apply

Obtain the deposited GNPS FBMN job inputs (metadata table in GNPS format, MZmine quantitative table restricted to 'Peak area', 'row m/z' and 'row retention time' columns, and the GNPS job identifier) and rerun the feature-based molecular networking job to regenerate the ion identity network. Count features, IIN groups, network nodes, singletons, and edges, then clean the GNPS molecular networking annotations against the spectral match results with ppm_error = 5, shared_peaks = 10, cosine = 0.7, and ionisation_mode = 'pos' (in the Celastraceae case, 2,494 library-annotated nodes reduced to 1,751 valid annotations). Optionally complement library annotations with SIRIUS/ISDB results filtered at min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0. Combine library and in-silico hits to compute the overall annotation rate across the feature set and write all statistics to a TSV file, checking the reproduced counts against the reported reference values.

## Related tools

- **GNPS** (Runs the Feature Based Molecular Networking job that produces the node, edge, IIN and spectral-annotation results being counted)
- **MZmine2** (Produces the feature quantitative table ('Peak area', 'row m/z', 'row retention time') used as FBMN input) — https://github.com/luigiquiros/inventa
- **MZmine3** (Alternative source of the quantitative table; the data_process_origin parameter must be set to 'MZMine3' if Ion Identity is used) — https://github.com/luigiquiros/inventa
- **SIRIUS** (Provides in-silico annotations (ISDB, CANOPUS) filtered at min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0 to complement library annotations)
- **timaR** (Supplies taxonomically reponderated in-silico annotation results used alongside GNPS spectral matches) — https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- **Inventa** (Implements the annotation cleanup step and computes annotation-related statistics over the feature table) — https://github.com/luigiquiros/inventa

## Examples

```
ppm_error = 5; shared_peaks = 10; cosine = 0.7; ionisation_mode = 'pos'
```

## Evaluation signals

- Reproduced counts match the reference values: 16,139 features, 14,554 IIN groups, 16,139 nodes, 5,922 singletons, 22,656 edges
- Library annotation count drops from 2,494 to 1,751 after cleaning with ppm_error = 5, shared_peaks = 10, cosine = 0.7, ionisation_mode = 'pos'
- Overall combined annotation rate falls near the reported ca. 68%
- Singleton count is consistent with node count (5,922 singletons out of 16,139 nodes) and edge count equals 22,656
- Summary statistics are written to a TSV file with all required columns present

## Limitations

- Reference counts are specific to the Celastraceae UHPLC-HRMS/MS dataset acquired in positive ionisation mode; other datasets or ionisation modes will yield different values
- The MZmine quantitative table must contain only 'Peak area' (or 'Peak height', one at a time), 'row m/z' and 'row retention time' columns; extra exported columns must be dropped manually or the quant_table() function adjusted
- In-silico annotation contribution depends on optional SIRIUS/ISDB and timaR inputs; without them only library-based annotation rates can be computed

## Evidence

- [methods] Clean up GNPS annotations thresholds: "ppm_error = 5                     # min error in ppm to consider an annotation valable"
- [methods] Feature Component ratio: "The **Feature Component (FC)** is a ratio of the number of specific non-annotated features over the total number of features of each extract."
- [readme] Quantitative table format: "MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns"
- [readme] TSV output: [evidence span withheld — anchor status: external_doc]
- [full_text] Annotation cleaning thresholds from reproduction workflow: [evidence span withheld — anchor status: not_found]
- [full_text] Reference statistics: [evidence span withheld — anchor status: not_found]
