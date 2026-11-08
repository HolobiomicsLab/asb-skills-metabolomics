---
name: fret-colocalization-cross-validation
description: Use when colocalization coefficients computed on two-channel CFP/YFP
  confocal images need independent validation against FRET measurements of the same
  cell groups, or when interpreting FRET-negative results for large complexes like
  CRM1 (130 kDa).
when_to_use_negative:
- When the candidate protein complex includes a large protein (e.g., CRM1 at 130 kDa)
  — FRET was negative for Rev-CRM1 despite colocalization, so FRET disagreement does
  not invalidate the colocalization result
- When the average distance between the binding proteins exceeds ~10 nm — FRET alone
  would only detect interaction below this distance, so it cannot serve as a validation
  standard for distant colocalization
- When analyzing live-cell dynamics (e.g., LMB-induced CRM1/Rev dissociation) where
  the goal is tracking proportions over time rather than validating group-level colocalization
  against FRET
tools:
- name: MIPAV
  role: Reference implementation of the Costes colocalization algorithm whose group-level
    output is cross-validated against FRET measurements
- name: Bitplane
  role: Commercial implementation of the same colocalization algorithm used to generate
    the coefficients being validated
- name: Zeiss LSM 510 confocal microscope
  role: Instrument used to acquire the two-channel 3D CFP/YFP image stacks of the
    three fixed HeLa groups analyzed and validated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:1f5c5e4044993de2587efd865909683d62a1700bacf3cc29a86842fcecf499b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fret-colocalization-cross-validation

## Summary

Cross-validates quantitative colocalization measurements (Costes block-scrambling algorithm) against independent FRET measurements on the same three groups of 40 fixed HeLa cells, establishing whether colocalization coefficients track physical protein-protein interaction.

## When to use

Use when colocalization coefficients computed on two-channel CFP/YFP confocal images need independent validation against FRET measurements of the same cell groups, or when interpreting FRET-negative results for large complexes like CRM1 (130 kDa).

## When NOT to use

- When the candidate protein complex includes a large protein (e.g., CRM1 at 130 kDa) — FRET was negative for Rev-CRM1 despite colocalization, so FRET disagreement does not invalidate the colocalization result
- When the average distance between the binding proteins exceeds ~10 nm — FRET alone would only detect interaction below this distance, so it cannot serve as a validation standard for distant colocalization
- When analyzing live-cell dynamics (e.g., LMB-induced CRM1/Rev dissociation) where the goal is tracking proportions over time rather than validating group-level colocalization against FRET

## Inputs

- Two-channel 3D confocal CFP/YFP image stacks for three fixed HeLa cotransfection groups (40 cells each), background-corrected
- Per-cell colocalization coefficients (Pearson correlation, proportion of protein colocalized) computed by the Costes algorithm
- Independent FRET measurements reported for the same three cell groups

## Outputs

- Group-level comparison of colocalization ordering and magnitudes versus FRET measurements
- Verification that the random-overlap control's coefficients fall within the random-overlap distribution
- Assessment of agreement/correlation between colocalization and FRET for known-interaction, co-compartmentalization, and random-overlap groups

## How to apply

Compute per-cell Pearson correlation and proportion-of-protein-colocalized coefficients on three fixed HeLa cotransfection groups (40 cells each: Rev-CFP + free YFP, Rev-CFP + Rev-YFP, Rev-CFP + Tat-YFP) using the block-scrambling algorithm (200 randomizations per region, blocks sized to the texture/PSF via the smaller autocorrelation width). First confirm the negative control: the free-YFP group's coefficients must fall within the random-overlap distribution, with pixel misclassification expected at the ~5% rate observed in simulations. Then compare the group-level ordering and magnitudes of colocalization coefficients against the FRET measurements reported for the same groups; the Rev-Rev and Rev-Tat groups should show significant colocalization while the random-overlap control does not. Note that FRET alone only detects interaction if the average distance between binding proteins is <10 nm, so agreement is expected at group level, not necessarily for every pair (e.g., FRET was negative for Rev-CRM1 despite colocalization).

## Related tools

- **MIPAV** (Reference implementation of the Costes colocalization algorithm whose group-level output is cross-validated against FRET measurements)
- **Bitplane** (Commercial implementation of the same colocalization algorithm used to generate the coefficients being validated)
- **Zeiss LSM 510 confocal microscope** (Instrument used to acquire the two-channel 3D CFP/YFP image stacks of the three fixed HeLa groups analyzed and validated)

## Evaluation signals

- The negative control (Rev-CFP + free YFP) colocalization coefficients must fall within the random-overlap distribution
- Pixel misclassification (false positives and false negatives) should be at the ~5% rate observed in simulations
- Group-level ordering and magnitudes of colocalization coefficients should match the ordering and magnitudes of the independent FRET measurements
- Rev-Rev and Rev-Tat groups must show significant colocalization while the random-overlap group shows only random overlap

## Limitations

- FRET alone would only detect interaction if the average distance between the binding proteins is <10 nm, so FRET-negative results do not imply absence of colocalization
- FRET was negative for the Rev-CRM1 complex, presumably due to the large size of CRM1 (130 kDa), illustrating that FRET validation fails for large complexes
- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization

## Evidence
