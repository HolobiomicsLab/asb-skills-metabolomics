---
name: reported-count-verification-against-reference-values
description: Use when a reported finding is a count of parameters grouped by sample
  type (EDTA plasma, serum, LiHep plasma) and delay type (pre- vs postcentrifugation)
  and you need to verify it against the PRIMA Panel stability time point data.
when_to_use_negative:
- When the reported claim is a qualitative finding (e.g., 'lactic acid was significantly
  affected by short delays') rather than a discrete count per sample type — use a
  per-parameter stability inspection instead.
- When raw NMR spectra or Bruker IVDr outputs are the only available input and stability
  time points have not yet been recomputed — the 240 min threshold cannot be applied
  without the stability time point table.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: PRIMA Panel
  role: Source of the stability time point data (Data tab) whose per-parameter time
    points in minutes, sorted by SPREC classification, are recomputed into per-sample-type
    counts for verification against reported values.
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Implementation language for loading the stability time point table, applying
    the 240 min threshold, and grouping counts by sample type and delay type.
  repo: https://github.com/funkam/PRIMA
- name: Shiny
  role: Framework hosting the PRIMA Panel app from which the recomputed stability
    time point table can be explored or exported before count verification.
  repo: https://github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:71632a62c2ee9b0062252493dd90e6afb2166a00794650e8604508dd2a8284e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reported-count-verification-against-reference-values

## Summary

Recompute tabular counts (e.g., number of metabolic parameters within a stability threshold per sample type and delay type) from the PRIMA Panel recomputed stability time point table and check each cell against reported reference values, accepting only when all cells match exactly.

## When to use

Use when a reported finding is a count of parameters grouped by sample type (EDTA plasma, serum, LiHep plasma) and delay type (pre- vs postcentrifugation) and you need to verify it against the PRIMA Panel stability time point data.

## When NOT to use

- When the reported claim is a qualitative finding (e.g., 'lactic acid was significantly affected by short delays') rather than a discrete count per sample type — use a per-parameter stability inspection instead.
- When raw NMR spectra or Bruker IVDr outputs are the only available input and stability time points have not yet been recomputed — the 240 min threshold cannot be applied without the stability time point table.

## Inputs

- Recomputed stability time point table: one row per metabolic parameter with stability time point in minutes, sample type (EDTA plasma, serum, LiHep), and delay type (pre- or postcentrifugation), sorted by SPREC classification
- Reported reference counts per delay type and sample type (e.g., precentrifugation: 26/26/15; postcentrifugation: 26/23/8 for EDTA plasma/serum/LiHep)

## Outputs

- Summary table with columns for delay type, sample type, and count of parameters within 4 h
- Match/mismatch flag per delay type × sample type cell
- Pass/fail verdict for the reported count claim

## How to apply

Load the recomputed stability time point table, in which each metabolic parameter's stability time point (in minutes) is annotated with its sample type (EDTA plasma, serum, LiHep) and delay type (pre- or postcentrifugation), sorted by SPREC classification. Convert the stability time points from minutes to hours and apply a 4 h (240 min) threshold to flag which parameters remain stable within the first 4 h for each parameter–sample type–delay type combination. Group the flagged rows by delay type and sample type, then count the distinct metabolic parameters with a stability time point ≤ 240 min in each of the six combinations. Assemble the counts into a summary table with columns for delay type, sample type, and count. Compare each computed count against the reported reference values and record a match/mismatch flag per cell; the verification is only accepted when all six counts equal the reported values exactly. Rationale: the article's stability time point concept marks when a metabolite's concentration changes by 20%, so the 240 min cutoff must be applied to the published time-point values rather than re-derived from raw spectra.

## Related tools

- **PRIMA Panel** (Source of the stability time point data (Data tab) whose per-parameter time points in minutes, sorted by SPREC classification, are recomputed into per-sample-type counts for verification against reported values.) — https://github.com/funkam/PRIMA
- **R** (Implementation language for loading the stability time point table, applying the 240 min threshold, and grouping counts by sample type and delay type.) — https://github.com/funkam/PRIMA
- **Shiny** (Framework hosting the PRIMA Panel app from which the recomputed stability time point table can be explored or exported before count verification.) — https://github.com/funkam/PRIMA

## Examples

```
Rscript -e 'tp <- read.csv("stability_time_points.csv"); tp$within4h <- tp$stability_min <= 240; aggregate(stability_min ~ delay_type + sample_type, data = tp[tp$within4h,], FUN = function(x) length(unique(x)))'
```

## Evaluation signals

- All six computed counts (delay type × sample type) equal the reported reference values exactly — e.g., precentrifugation 26 (EDTA plasma), 26 (serum), 15 (LiHep); postcentrifugation 26 (EDTA plasma), 23 (serum), 8 (LiHep).
- Every metabolic parameter is counted at most once per delay type × sample type cell (no duplicates after grouping).
- Stability time points are compared in a consistent unit — verify that minutes-to-hours conversion was applied before applying the 240 min threshold.
- The summary table contains exactly the three columns: delay type, sample type, count.
- Any mismatched cell produces an explicit mismatch flag rather than being silently reported as a match.

## Limitations

- Exact-match acceptance means a single off-by-one count in any of the six cells fails the whole verification, even if the discrepancy is small.
- The skill depends on the stability time point table being annotated with sample type and delay type; if the SPREC classification sorting is missing, grouping may be ambiguous.
- No version information is provided for the PRIMA Panel, so the exact stability time point values used for verification may not be reproducible across app versions.

## Evidence

- [readme] stability time points in minutes sorted by SPREC classification, annotated with sample type and delay type: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] data split by pre- and post-centrifugation delays: "The data is split according the two different delays (pre- and post-centrifugation"
- [abstract] concept of stability time point at 20% change: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [abstract] PRIMA Panel is an open-source R Shiny tool: "the authors developed the PRIMA Panel, an open-source R Shiny tool"
- [readme] performance report thresholds adjustable per metabolite: "The colors, as well as the % threshholds, can be adjusted. The table can be downloaded as a .HTML report or the table directly as .csv (or other formats)."
