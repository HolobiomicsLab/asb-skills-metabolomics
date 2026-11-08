---
name: diagnostic-fragment-substructure-annotation
description: Use when LC-HRMS/MS feature tables contain precursors whose exact masses
  match no catalogued PFAS but whose MS/MS may carry fluorine-containing diagnostic
  fragments, and the goal is flagging potential unknown PFAS.
when_to_use_negative:
- Precursor exact masses already match catalogued PFAS — use the exact mass matching
  to catalogued PFAS step instead of reverse fragment screening
- Analysis target is lipids rather than PFAS — the FluoroMatch code requires toggling
  Lipid <- FALSE so the framework defaults to PFAS analysis
- Precursors lack MS/MS spectra or fall outside the PFAS mass-defect window (−0.116
  to 0.268) and m/z 117–1189 range — fragment screening will not be able to assign
  fluorine-containing substructures
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: FluoroMatch 2.0
  role: Software implementing the non-targeted PFAS identification workflow that includes
    fragment screening for assigning fluorine-containing fragments and substructures
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular R framework in which the fragment-screening module is run with FLOW
    <- FALSE, loading the feature table and 777-fragment library
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: Full workflow software covering all PFAS discovery data-processing steps of
    which fragment screening is one stage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:c16b422888588ee46d09d58a9c1c04aebefd7a78b7180e64f7558318ca472e0f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# diagnostic-fragment-substructure-annotation

## Summary

A reverse fragment screening skill that searches precursor MS/MS spectra against a reconstructed library of 777 fluorine-containing PFAS fragments in FluoroMatch 2.0, assigning PFAS substructures to precursors with no exact mass match and flagging potential unknown PFAS via diagnostic fragments such as [SF5]− and [SO3F]−.

## When to use

Use when LC-HRMS/MS feature tables contain precursors whose exact masses match no catalogued PFAS but whose MS/MS may carry fluorine-containing diagnostic fragments, and the goal is flagging potential unknown PFAS.

## When NOT to use

- Precursor exact masses already match catalogued PFAS — use the exact mass matching to catalogued PFAS step instead of reverse fragment screening
- Analysis target is lipids rather than PFAS — the FluoroMatch code requires toggling Lipid <- FALSE so the framework defaults to PFAS analysis
- Precursors lack MS/MS spectra or fall outside the PFAS mass-defect window (−0.116 to 0.268) and m/z 117–1189 range — fragment screening will not be able to assign fluorine-containing substructures

## Inputs

- Blank-filtered feature table produced by the upstream feature blank-filtering step (LC-HRMS/MS data, e.g. AFFF samples)
- Reconstructed library of 777 fluorine-containing PFAS fragments
- Precursor m/z and MS/MS spectra (tandem mass spectrometry) for each feature

## Outputs

- fragment_assignments.csv — per-feature fragment/substructure assignment table
- List of precursors flagged as potential PFAS despite no exact mass match, including unsaturated-PFAS candidates bearing [SF5]−/[SO3F]− diagnostic fragments

## How to apply

Load the blank-filtered feature table from the upstream blank-filtering step into the FluoroMatch Modular fragment-screening module, together with the reconstructed library of 777 fluorine-containing fragments. Restrict precursor candidates to the PFAS mass-defect window of −0.116 to 0.268 (covering 98% of the EPA PFAS structure list of over 8000 PFAS) and m/z 117–1189. Reverse-screen each precursor's MS/MS against the 777 fluorine-containing fragments so that matched fragments assign PFAS substructures to each feature, even when the precursor exact mass has no match. Flag features carrying the pentafluorosulfide ([SF5]−) and sulfonate ([SO3F]−) diagnostic fragments as unsaturated-PFAS candidates, such as the proposed new AFFF PFAS class with a pentafluorosulfide and sulfonic acid terminus (C8 candidate, Fig. 7). Write the per-feature fragment/substructure assignments to fragment_assignments.csv, leaving chemical plausibility of novel classes to expert review.

## Related tools

- **FluoroMatch 2.0** (Software implementing the non-targeted PFAS identification workflow that includes fragment screening for assigning fluorine-containing fragments and substructures) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular R framework in which the fragment-screening module is run with FLOW <- FALSE, loading the feature table and 777-fragment library) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (Full workflow software covering all PFAS discovery data-processing steps of which fragment screening is one stage)

## Examples

```
FLOW <- FALSE; csvInput <- FALSE; ManuallyInputVariables <- FALSE; Lipid <- FALSE; source("Modular.r")
```

## Evaluation signals

- fragment_assignments.csv exists and parses as a well-formed table with per-feature fragment/substructure assignments
- The C8 candidate row in fragment_assignments.csv contains [SF5]− and [SO3F]− diagnostic fragment assignments
- Screened precursors fall within the PFAS mass-defect range −0.116 to 0.268 and m/z 117–1189
- Downstream filtering removes a large majority of features likely not PFAS (the full FluoroMatch Flow 2 pipeline reported filtering out 96% of features which were likely not PFAS)
- Novel flagged classes (e.g. pentafluorosulfide/sulfonic acid terminus candidates) are routed to expert review rather than auto-accepted

## Limitations

- Chemical plausibility of newly proposed PFAS classes (e.g. the pentafluorosulfide and sulfonic acid terminus class) is left to expert review; fragment matching alone does not confirm structure
- The mass-defect window −0.116 to 0.268 covers 98% of the EPA PFAS list, so rare PFAS outside this window may be missed
- Assignments are only as good as the reconstructed 777 fluorine-containing fragment library; fragments absent from the library cannot be matched

## Evidence
