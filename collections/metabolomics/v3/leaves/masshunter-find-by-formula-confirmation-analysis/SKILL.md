---
name: masshunter-find-by-formula-confirmation-analysis
description: Use when validating suspect-screening annotations against an independent
  formula-confirmation tool — e.g. tabulating FbF confirmation rates for B vs B− suspects
  or counting disagreements with level-A assignments.
when_to_use_negative:
- Input has not passed through the FluoroMatch annotation tiers — without B/B− and
  level-A assignment labels there is nothing to cross-tabulate against FbF outcomes.
- The goal is de novo PFAS discovery or feature filtering rather than validating already-annotated
  suspects (use the FluoroMatch Flow 2.0 filtering workflow instead).
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FluoroMatch 2.0
  role: source of the in-silico fragment-based (B/B−) suspect annotations being validated
    against level-A assignments and FbF confirmation
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: modular code base of FluoroMatch 2.0 in which the annotation workflow producing
    the suspects is executed
  repo: github.com/InnovativeOmics/Core-Match
- name: Agilent MassHunter Find-by-Formula
  role: external formula-confirmation benchmark classifying each suspect as confident,
    tentative, or not found
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:a7255d0de3fb2c667f0b922df1b56705232ad8d5e19f16624c556ec0e5f4ab04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Agilent MassHunter Find-by-Formula
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# masshunter-find-by-formula-confirmation-analysis

## Summary

Benchmark the reliability of in-silico fragment-based suspect annotations (level B and B−) from FluoroMatch 2.0 by tallying disagreements against level-A assignments and classifying Agilent MassHunter Find-by-Formula confirmation outcomes (confident / tentative / not found), with isotope-pattern checks to expose false positives.

## When to use

Use when validating suspect-screening annotations against an independent formula-confirmation tool — e.g. tabulating FbF confirmation rates for B vs B− suspects or counting disagreements with level-A assignments.

## When NOT to use

- Input has not passed through the FluoroMatch annotation tiers — without B/B− and level-A assignment labels there is nothing to cross-tabulate against FbF outcomes.
- The goal is de novo PFAS discovery or feature filtering rather than validating already-annotated suspects (use the FluoroMatch Flow 2.0 filtering workflow instead).

## Inputs

- FluoroMatch 2.0 annotation output table containing in-silico fragment-based (B and B−) suspect annotations
- ESM validation tables with Agilent MassHunter Find-by-Formula confirmation results
- Level-A assignment list used as the benchmark reference

## Outputs

- validation_statistics.csv containing disagreement counts (e.g. 3/58) and confirmation-rate fractions per annotation tier
- Flagged list of false-positive suspect series (e.g. bromine-containing PFAS lacking the Br isotope pattern)

## How to apply

Load the FluoroMatch 2.0 annotation output alongside the ESM validation tables containing Agilent MassHunter Find-by-Formula results. First, compare the 58 in-silico fragment-based (B/B−) suspect annotations against the level-A assignments and tally disagreements, which should reproduce the reported 3-of-58 (5%) disagreement rate. Then classify each suspect by the Find-by-Formula confirmation category, separating B suspects (78% confident, 11% tentative, 11% not found) from B− suspects (33% confident, 26% tentative, 40% not found) so the two confidence tiers are not pooled. Inspect homologous series annotations for diagnostic isotope evidence — e.g. a B-series of bromine-containing PFAS that lacks the characteristic Br isotope pattern is a false positive and must be flagged. Finally, write the disagreement counts and confirmation-rate fractions to validation_statistics.csv; the interpretation of the Br false positive is reserved for expert review, not automated.

## Related tools

- **FluoroMatch 2.0** (source of the in-silico fragment-based (B/B−) suspect annotations being validated against level-A assignments and FbF confirmation) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (modular code base of FluoroMatch 2.0 in which the annotation workflow producing the suspects is executed) — github.com/InnovativeOmics/Core-Match
- **Agilent MassHunter Find-by-Formula** (external formula-confirmation benchmark classifying each suspect as confident, tentative, or not found)

## Examples

```
Rscript Modular.r  # with FLOW <- FALSE; csvInput <- FALSE; Lipid <- FALSE (defaults to PFAS analysis)
```

## Evaluation signals

- Disagreement tally between B/B− suspects and level-A assignments must equal the reported 3-of-58 (5%) rate.
- B-suspect FbF confirmation categories must reproduce 78% confident / 11% tentative / 11% not found exactly.
- B− suspect FbF categories must reproduce 33% confident / 26% tentative / 40% not found exactly.
- validation_statistics.csv must contain a separate field for each reported confirmation category (confident, tentative, not found), not a single pooled rate.
- The bromine-containing false-positive series must be present as a flagged record reserved for expert review.

## Limitations

- B− (lower-confidence) suspect annotations are weakly corroborated by FbF: 40% were 'not found' and 26% tentative, so B− results should not be reported as confirmed.
- Homologous-series annotation can produce false positives detectable only by isotope evidence (e.g. absent Br isotope pattern), which requires expert review to interpret.
- The validation table must match reported reference values exactly; partial or pooled reporting is not acceptable for reproducibility.

## Evidence
