---
name: accuracy-rate-calculation
description: Use when a benchmark set of authentic-standard (MSI level 1) MS/MS spectra
  has been run through an in silico annotation tool such as CSI:FingerID and the top-ranked
  candidate agreement rate must be quantified against the known identities.
when_to_use_negative:
- Input spectra are only annotated at MSI level 2 or 3 (precursor mass / in silico
  matches) — there is no authentic-standard identity to score against.
- You want to annotate unknown spectra rather than benchmark annotation accuracy —
  use CSI:FingerID annotation directly instead of the accuracy calculation.
- Search space or mass tolerance differs from the published benchmark (e.g. not the
  biodatabase at 5 ppm), making rates incomparable to the 40%/52% reference values.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Sirius/CSI:Finger ID
  role: Generates the top-ranked candidate structures for each MSI level 1 MS/MS spectrum
    whose agreement with authentic-standard identities is scored
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: In silico fragmentation / structure search tool whose top-hit predictions
    are the objects of the accuracy calculation
  repo: https://github.com/sirius-ms/sirius
- name: HILIC-Q Exactive HF mass spectrometry
  role: Instrument platform that acquired the MSI level 1 benchmark MS/MS spectra
    being scored
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:ddfac85c657de61e42b34419c906d81ec4906f19a4c95e1b0379e6d0c1117a75
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - HILIC-Q Exactive HF mass spectrometry
  - Sirius/CSI:Finger ID
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# accuracy-rate-calculation

## Summary

Computes the fraction of MSI level 1 (authentic-standard) MS/MS spectra whose top-ranked (and top-3) CSI:FingerID structure predictions agree with the known identity, benchmarking in silico structure annotation accuracy as 40% top-hit / 52% top-3 on this urine HILIC-Q Exactive dataset.

## When to use

Use when a benchmark set of authentic-standard (MSI level 1) MS/MS spectra has been run through an in silico annotation tool such as CSI:FingerID and the top-ranked candidate agreement rate must be quantified against the known identities.

## When NOT to use

- Input spectra are only annotated at MSI level 2 or 3 (precursor mass / in silico matches) — there is no authentic-standard identity to score against.
- You want to annotate unknown spectra rather than benchmark annotation accuracy — use CSI:FingerID annotation directly instead of the accuracy calculation.
- Search space or mass tolerance differs from the published benchmark (e.g. not the biodatabase at 5 ppm), making rates incomparable to the 40%/52% reference values.

## Inputs

- 103 unambiguously assigned MSI level 1 MS/MS spectra from authentic standards (urine, HILIC-Q Exactive HF platform)
- Authentic-standard identity table (compound name per spectrum, MSI confidence level 1 assignments)
- CSI:FingerID candidate result lists (top-ranked structures per spectrum, biodatabase search, 5 ppm tolerance)

## Outputs

- Top-hit accuracy rate (fraction of spectra whose rank-1 CSI:FingerID candidate matches the authentic-standard identity; published value 40%, 41/103)
- Top-3 accuracy rate (fraction with correct structure in top 3 candidates; published value 52%)
- Per-compound comparison table recording predicted structure, known identity, and rank position

## How to apply

Load the unambiguously assigned MSI level 1 spectra (103 compounds acquired on the HILIC-Q Exactive platform in the reference study) and submit each through the SIRIUS pipeline to CSI:FingerID using the biodatabase as the structure search space with a 5 ppm precursor mass tolerance. Retrieve the top-ranked candidate structures from the CSI:FingerID web service for every spectrum. For each compound, compare the top-ranked (and top-3) predicted structures against the authentic-standard assignment, recording exact structural agreement and rank position. Summarize agreement rates as top-hit accuracy (correct/total, e.g. 41/103 = 40%) and top-3 accuracy (52%). Accept the analysis only when the recomputed rates reproduce the published 40% top-ranked and 52% top-3 values for the MSI level 1 compound set.

## Related tools

- **Sirius/CSI:Finger ID** (Generates the top-ranked candidate structures for each MSI level 1 MS/MS spectrum whose agreement with authentic-standard identities is scored) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (In silico fragmentation / structure search tool whose top-hit predictions are the objects of the accuracy calculation) — https://github.com/sirius-ms/sirius
- **HILIC-Q Exactive HF mass spectrometry** (Instrument platform that acquired the MSI level 1 benchmark MS/MS spectra being scored)

## Evaluation signals

- Recomputed top-hit accuracy equals the published 40% (41 of 103 compounds) for the MSI level 1 set.
- Recomputed top-3 accuracy equals the published 52% for the MSI level 1 set.
- Denominator of the rate is exactly the 103 unambiguously assigned MSI level 1 spectra (not the full 175 MSI level 1 identifications).
- Every per-compound record contains both the authentic-standard (MSI level 1) identity and the rank position of the matching CSI:FingerID candidate.
- All spectra were searched against the biodatabase with a 5 ppm precursor mass tolerance before scoring.

## Limitations

- Structure assignments from Sirius/CSI:Finger ID require validation with authentic chemicals; the rate is only meaningful against an MSI level 1 benchmark.
- Accuracy is database- and tolerance-dependent (biodatabase, 5 ppm) — rates will shift with different search spaces.
- The article notes the field lacks generally accepted, validated automated confidence-level calculations with false-discovery rate assessments in metabolomics MS/MS, so this rate is a bespoke benchmark rather than a standardized metric.
- Only 103 of the 175 MSI level 1 identifications were unambiguously assignable for this benchmark, so coverage of the full MSI level 1 set is partial.

## Evidence

- [abstract] Top-hit accuracy reference value: "Testing the top-ranked metabolites in CSI:Finger ID annotations yielded 40% accuracy when applied to the MSI level 1 identified compounds"
- [abstract] Benchmark source spectra: "175 compounds using authentic standards with precursor mass, retention time, and MS/MS matching"
- [abstract] Annotation tool under test: "we used the in silico fragmentation tool CSI:FingerID and the new NIST hybrid search to annotate all further compounds (MSI level 3)"
- [readme] SIRIUS framework used to run CSI:FingerID: [evidence span withheld — anchor status: external_doc]
