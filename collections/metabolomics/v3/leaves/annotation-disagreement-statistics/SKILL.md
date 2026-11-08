---
name: annotation-disagreement-statistics
description: Use when a FluoroMatch 2.0 (or Modular) annotation output contains suspect-level
  B/B− assignments that must be benchmarked against trusted level-A assignments and
  Find-by-Formula results before reporting.
when_to_use_negative:
- The goal is to generate new B/B− annotations rather than to benchmark existing FluoroMatch
  output — use FluoroMatch 2.0's annotation workflow instead
- No independent reference exists (neither level-A assignments nor Find-by-Formula
  results), so no disagreement or confirmation statistics can be computed
- The input is already a summary of annotation reliability rather than raw per-feature
  annotations, so tallying would double-count
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: FluoroMatch 2.0
  role: Source of the in silico fragment-based (B/B−) suspect annotations being validated;
    its confidence-level framework defines the annotation tiers compared
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular distribution used to run/inspect the FluoroMatch workflow that produced
    the annotations under validation
  repo: github.com/InnovativeOmics/Core-Match
- name: Agilent MassHunter Find-by-Formula
  role: Independent formula-confirmation benchmark whose confident/tentative/not-found
    categories are tallied against B and B− suspects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:9ac5149ac15ffc20a7b61754cf4e6ddb152189a227587dabff87d0f13a8d7361
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Agilent MassHunter Find-by-Formula
  - FluoroMatch 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# annotation-disagreement-statistics

## Summary

Validates FluoroMatch 2.0 in silico fragment-based (B and B−) PFAS annotations by tallying disagreements against level-A assignments and Agilent MassHunter Find-by-Formula confirmation categories, so overreporting is quantified rather than assumed absent.

## When to use

Use when a FluoroMatch 2.0 (or Modular) annotation output contains suspect-level B/B− assignments that must be benchmarked against trusted level-A assignments and Find-by-Formula results before reporting.

## When NOT to use

- The goal is to generate new B/B− annotations rather than to benchmark existing FluoroMatch output — use FluoroMatch 2.0's annotation workflow instead
- No independent reference exists (neither level-A assignments nor Find-by-Formula results), so no disagreement or confirmation statistics can be computed
- The input is already a summary of annotation reliability rather than raw per-feature annotations, so tallying would double-count

## Inputs

- FluoroMatch 2.0 annotation output table with confidence-level codes (A, B, B−) per suspect feature
- ESM validation tables containing Agilent MassHunter Find-by-Formula confirmation results (confident / tentative / not found)
- Level-A assignment list used as the ground-truth reference
- MS1 isotope-pattern data for suspect series (to check for the characteristic Br isotope pattern)

## Outputs

- validation_statistics.csv containing the disagreement tally (e.g. 3 of 58), total compared assignments, and one field per Find-by-Formula confirmation category for B and B− suspects
- Flagged list of suspected false-positive series (e.g. the bromine-containing PFAS series lacking the Br isotope pattern), marked for expert review

## How to apply

Load the FluoroMatch 2.0 annotation output together with the ESM validation tables holding Agilent MassHunter Find-by-Formula results. Restrict the comparison to the in silico fragment-based suspects (levels B and B−) and tally cases where they conflict with level-A assignments, reproducing the paper's reference rate of 3 disagreements in 58 assignments (≈5%). Separately, classify each suspect by Find-by-Formula confirmation outcome — confident, tentative, or not found — and compute the fractions (reference: B suspects 78%/11%/11%; B− suspects 33%/26%/40%). Inspect any suspect series whose formula implies bromine but whose MS1 data lack the characteristic Br isotope pattern, flag it as a false positive, and reserve that interpretation for expert review rather than auto-rejecting. Write the disagreement count, total, and each confirmation-category fraction to validation_statistics.csv with one field per reported category.

## Related tools

- **FluoroMatch 2.0** (Source of the in silico fragment-based (B/B−) suspect annotations being validated; its confidence-level framework defines the annotation tiers compared) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular distribution used to run/inspect the FluoroMatch workflow that produced the annotations under validation) — github.com/InnovativeOmics/Core-Match
- **Agilent MassHunter Find-by-Formula** (Independent formula-confirmation benchmark whose confident/tentative/not-found categories are tallied against B and B− suspects)

## Evaluation signals

- Disagreement tally between B/B− annotations and level-A assignments equals exactly 3 of 58 (≈5%)
- Find-by-Formula confirmation fractions for B suspects equal 78% confident / 11% tentative / 11% not found
- Find-by-Formula confirmation fractions for B− suspects equal 33% confident / 26% tentative / 40% not found
- validation_statistics.csv contains a dedicated field for every reported confirmation category (confident, tentative, not found) for both B and B−
- The bromine-containing false-positive series (absent Br isotope pattern) is flagged but its interpretation is deferred to expert review, not auto-excluded

## Limitations

- B− suspects show much weaker independent confirmation (33% confident, 40% not found) than B suspects, so statistics pooled over both tiers are misleading
- Isotope-pattern-based exclusion (e.g. missing Br pattern) is an interpretive judgment that the workflow reserves for expert review, not an automated rejection rule
- Disagreement rates are benchmark-specific: they depend on the level-A assignments and Find-by-Formula settings used, and do not transfer to other reference sets

## Evidence

- [abstract] confidence-level criterion reduces overreporting: "a comprehensive confidence level criterion is implemented to help users understand annotation certainty and integrate various layers of evidence to reduce overreporting"
- [abstract] fragment screening is an evidence layer feeding annotations: "retention time pattern analysis, class-based MS/MS screening, fragment screening, and predicted MS/MS from SMILES structures"
- [readme] README: software defaults to PFAS analysis unless toggled: "Make sure to toggle the following parameters depending on your application, if both are FALSE it defaults to PFAS analysis"
- [readme] README: modular distribution and manual locations: [evidence span withheld — anchor status: external_doc]
