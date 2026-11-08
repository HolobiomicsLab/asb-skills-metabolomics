---
name: metabolite-standard-recovery-evaluation
description: Use when a new annotation or feature-grouping tool must be quantitatively
  validated against standards of known identity and mass, and a per-compound recovery
  rate is needed.
when_to_use_negative:
- Input data are study samples with unknown compounds — there is no ground truth to
  compute recovery against; use a separate standard mixture run.
- The goal is evaluating peak-detection sensitivity (e.g. how many true features XCMS
  finds) rather than whether downstream annotation correctly groups and labels each
  analyte.
- Standards elute too early for the pipeline's retention-time filter — CAMERA excluded
  compounds below 45 s, so a recovery test over such standards would conflate filtering
  with annotation quality.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Annotates compound spectra (isotopes, adducts, quasi-molecular ion and accurate
    mass) for the standard-mixture run; its outputs are what the recovery check inspects
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection producing the peak lists CAMERA consumes when the
    recovery run is executed
- name: MM39 standard mixture
  role: The 39-compound benchmark mixture (161–822 Da) whose recovery rate is the
    evaluation metric
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:b21289457262a64730d4d312d8eeffe76fda3555983f5cef8767ad6859a1579e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - MM39 standard mixture
  - XCMS
  - findIsotopes
  - getPeaklist
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metabolite-standard-recovery-evaluation

## Summary

Benchmark a feature-annotation pipeline by running it over a defined chemical standard mixture and measuring how many of the known standards are recovered with correct quasi-molecular-ion annotation, as CAMERA's authors did with a 39-compound mixture (MM39).

## When to use

Use when a new annotation or feature-grouping tool must be quantitatively validated against standards of known identity and mass, and a per-compound recovery rate is needed.

## When NOT to use

- Input data are study samples with unknown compounds — there is no ground truth to compute recovery against; use a separate standard mixture run.
- The goal is evaluating peak-detection sensitivity (e.g. how many true features XCMS finds) rather than whether downstream annotation correctly groups and labels each analyte.
- Standards elute too early for the pipeline's retention-time filter — CAMERA excluded compounds below 45 s, so a recovery test over such standards would conflate filtering with annotation quality.

## Inputs

- LC/MS raw or peak-listed data for a known standard mixture (e.g. 39 compounds, 161–822 Da)
- Compound list with sum formulas, molar masses and identifiers (e.g. PubChem IDs) for the standards
- Reference annotation or prior identification for each standard

## Outputs

- Recovery rate per ionization mode (e.g. 89.7% positive, 90.3% negative)
- Per-standard classification of recovered / missed with failure-reason taxonomy
- Counts of compound spectra with only molecular+fragment ions (unannotatable without extra neutral-loss rules)

## How to apply

Prepare a mixture of known compounds spanning the mass range of interest — CAMERA's authors used MM39, 39 standards covering 161–822 Da — and acquire it with the same LC/MS protocol as the study samples (UPLC/ESI-QTOF, m/z 100–1000, 3 Hz). Run the same detection, grouping and annotation workflow used for the real samples over the standard data, then check, for each standard, whether the pipeline produced a compound spectrum with the correct quasi-molecular ion and proposed accurate mass. Tally recovered vs. expected standards per polarity (CAMERA reported 89.7% positive, 90.3% negative) and separately diagnose the misses: spectra with too few explained features, ties in precedence scores, or spectra containing only molecular and fragment ions with no adducts. Report recovery rate as the headline metric and the miss taxonomy as the diagnostic.

## Related tools

- **CAMERA** (Annotates compound spectra (isotopes, adducts, quasi-molecular ion and accurate mass) for the standard-mixture run; its outputs are what the recovery check inspects) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection producing the peak lists CAMERA consumes when the recovery run is executed)
- **MM39 standard mixture** (The 39-compound benchmark mixture (161–822 Da) whose recovery rate is the evaluation metric)

## Examples

```
For the evaluation we used a mixture of 39 known compounds (short: MM39)
```

## Evaluation signals

- Recovery rate computed separately per polarity and reported to one decimal (CAMERA: 89.7% positive, 90.3% negative).
- Every standard in the input list maps to at most one annotated compound spectrum; multi-mapping or zero-mapping cases are logged, not silently dropped.
- Misses are decomposed into named failure modes — too few explained features, tied precedence scores, only molecular+fragment ions — rather than lumped as 'failed'.
- No ad-hoc heuristics (e.g. 'most intense feature = [M+H]+') are used to force recovery on marginal spectra.
- The mass range and retention-time coverage of the standard mixture are reported so the recovery rate can be contextualized.

## Limitations

- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated unless the corresponding neutral loss is added to the rule table.
- Tied precedence scores across several hypotheses prevent a unique quasi-molecular-ion call even when the spectrum is otherwise informative.
- Sample concentration must avoid MS detector saturation, which shifts m/z and retention-time centroids and truncates peak profiles, corrupting the recovery measurement.
- Recovery rate depends on the mass and retention-time window of the standard set; extending it outside the validated range (e.g. below 45 s) is not covered by the published benchmark.

## Evidence
