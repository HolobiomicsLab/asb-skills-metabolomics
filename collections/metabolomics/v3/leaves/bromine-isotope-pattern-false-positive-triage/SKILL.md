---
name: bromine-isotope-pattern-false-positive-triage
description: Use when auditing FluoroMatch 2.0 suspect-level B/B− annotations against
  trusted level-A assignments or Find-by-Formula results, especially when suspect
  formulas contain heteroatoms (Br, Cl) whose isotope envelopes can be checked in
  MS1 data.
when_to_use_negative:
- Annotations already at level-A with corroborated MS/MS evidence — these are the
  reference set, not the audit target.
- Datasets where only MS/MS spectra are available without MS1 isotope-resolution data,
  since the Br isotope-pattern check cannot be performed.
- General PFAS discovery runs (feature detection, blank filtering, homologous series
  detection) — this skill applies only to post-annotation validation.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: FluoroMatch 2.0
  role: Source of the in silico fragment-based (B/B−) suspect annotations that are
    audited and triaged for false positives
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular workflow mode of the FluoroMatch distribution used to run the annotation
    pipeline whose outputs feed this validation step
  repo: github.com/InnovativeOmics/Core-Match
- name: Agilent MassHunter Find-by-Formula
  role: External formula-confirmation engine providing confident/tentative/not-found
    categories used to benchmark B and B− suspects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:504b7ecd0ac57de4cbb41852fe073b61ad7cea494642037ba39e3b01e359fb22
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Agilent MassHunter Find-by-Formula
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bromine-isotope-pattern-false-positive-triage

## Summary

A validation skill that triages in silico fragment-based PFAS suspect annotations (B and B− levels from FluoroMatch 2.0) by benchmarking them against level-A assignments and Agilent MassHunter Find-by-Formula confirmation, and uses absent isotope evidence — e.g. a missing Br isotope pattern in an annotated bromine-containing PFAS series — to expose false positives.

## When to use

Use when auditing FluoroMatch 2.0 suspect-level B/B− annotations against trusted level-A assignments or Find-by-Formula results, especially when suspect formulas contain heteroatoms (Br, Cl) whose isotope envelopes can be checked in MS1 data.

## When NOT to use

- Annotations already at level-A with corroborated MS/MS evidence — these are the reference set, not the audit target.
- Datasets where only MS/MS spectra are available without MS1 isotope-resolution data, since the Br isotope-pattern check cannot be performed.
- General PFAS discovery runs (feature detection, blank filtering, homologous series detection) — this skill applies only to post-annotation validation.

## Inputs

- FluoroMatch 2.0 annotation output containing in silico fragment-based B and B− suspect assignments
- ESM validation tables with Agilent MassHunter Find-by-Formula confirmation results (confident / tentative / not found)
- Level-A assignment list used as the reference annotation benchmark
- MS1 accurate-mass data with isotope envelopes for the flagged suspect features

## Outputs

- validation_statistics.csv with disagreement counts and confirmation-rate fractions per suspect level
- Flagged list of false-positive suspect series (e.g. the bromine-containing B-series lacking the Br isotope pattern)
- Per-category confirmation tally fields (confident / tentative / not found) for expert review

## How to apply

Load the FluoroMatch 2.0 annotation output and the ESM validation tables containing Agilent MassHunter Find-by-Formula results, then compare the 58 in silico fragment-based (B/B−) suspect annotations against level-A assignments to tally disagreements (expected 3 of 58, ~5%). Next, tally Find-by-Formula confirmation categories separately for the two suspect levels: B suspects should reproduce 78% confident / 11% tentative / 11% not found, while B− suspects should reproduce 33% confident / 26% tentative / 40% not found — the markedly lower B− confirmation rate is itself a signal that lower-level annotations need isotope-level scrutiny. For any suspect series annotated as bromine-containing PFAS, inspect the MS1 isotope envelope: absence of the characteristic ~1:1 M/M+2 Br isotope pattern in a B-series annotation is grounds to mark that series as a proven false positive. Write the disagreement statistics and confirmation-rate fractions to validation_statistics.csv, flag the bromine-containing false-positive series, and reserve the final isotope-based rejection interpretation for expert review rather than automated deletion.

## Related tools

- **FluoroMatch 2.0** (Source of the in silico fragment-based (B/B−) suspect annotations that are audited and triaged for false positives) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular workflow mode of the FluoroMatch distribution used to run the annotation pipeline whose outputs feed this validation step) — github.com/InnovativeOmics/Core-Match
- **Agilent MassHunter Find-by-Formula** (External formula-confirmation engine providing confident/tentative/not-found categories used to benchmark B and B− suspects)

## Examples

```
Rscript -e 'FLOW <- FALSE; csvInput <- TRUE; ManuallyInputVariables <- FALSE; Lipid <- FALSE; TWeen_pos <- FALSE'  # Modular mode defaults to PFAS analysis
```

## Evaluation signals

- Disagreement tally between B/B− annotations and level-A assignments must reproduce 3 of 58 (5%) exactly.
- B suspect confirmation rates must match 78% confident / 11% tentative / 11% not found.
- B− suspect confirmation rates must match 33% confident / 26% tentative / 40% not found.
- The output table must contain an explicit field for each confirmation category and a reserved field for the Br false-positive interpretation.
- The bromine-containing B-series flagged as false positive must be accompanied by evidence of the absent Br isotope pattern, not by automated deletion.

## Limitations

- The isotope-based false-positive interpretation of the bromine-containing series is reserved for expert review and should not be automated as a deletion rule.
- B− suspect annotations have substantially lower external confirmation (33% confident, 40% not found), so conclusions drawn from B− alone are unreliable.
- The 3/58 disagreement rate and confirmation fractions are dataset- and instrument-specific (Agilent MassHunter Find-by-Formula) and may not transfer to other benchmark sets.

## Evidence
