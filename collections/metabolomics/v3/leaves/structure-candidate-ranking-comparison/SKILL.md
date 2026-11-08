---
name: structure-candidate-ranking-comparison
description: Use when you have MS/MS spectra of compounds already assigned at MSI
  confidence level 1 via authentic standards and need to measure how often an in silico
  annotator (CSI:FingerID) ranks the true structure first or in the top 3.
when_to_use_negative:
- The input spectra are not supported by authentic-standard measurements — results
  would only reach MSI level 2/3 confidence, not the level 1 ground truth this benchmark
  requires
- You lack a defined structure search space or mass tolerance, so candidate ranking
  is not reproducible against the published 40%/52% figures
- The goal is general chemical-class profiling of unannotated spectra (ClassyFire/ontology
  work), not accuracy measurement of structure ranking
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: CSI:FingerID
  role: In silico fragmentation tool that returns ranked candidate structures; its
    top-ranked predictions are the subject of the accuracy comparison against MSI
    level 1 compounds
  repo: github.com/sirius-ms/sirius
- name: Sirius/CSI:Finger ID
  role: SIRIUS pipeline (CLI/GUI/web service) that submits fragmentation trees and
    spectra to CSI:FingerID and retrieves candidate rankings
  repo: github.com/sirius-ms/sirius
- name: HILIC-Q Exactive HF mass spectrometry
  role: Instrument platform that acquired the MSI level 1 MS/MS spectra being benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:dcd7483be0e7da6b938ea5d2998bd82daa79a6423732d735eb0045e9f1caccc3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CSI:FingerID
  - HILIC-Q Exactive HF mass spectrometry
  - Sirius/CSI:Finger ID
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# structure-candidate-ranking-comparison

## Summary

Benchmarks in silico structure annotation (CSI:FingerID) top-ranked and top-3 candidate predictions against authentic-standard (MSI level 1) compound assignments to quantify annotation accuracy, as in the urine HILIC/CSH Q Exactive HF study where 40% of 103 MSI level 1 compounds were correctly annotated as the top hit.

## When to use

Use when you have MS/MS spectra of compounds already assigned at MSI confidence level 1 via authentic standards and need to measure how often an in silico annotator (CSI:FingerID) ranks the true structure first or in the top 3.

## When NOT to use

- The input spectra are not supported by authentic-standard measurements — results would only reach MSI level 2/3 confidence, not the level 1 ground truth this benchmark requires
- You lack a defined structure search space or mass tolerance, so candidate ranking is not reproducible against the published 40%/52% figures
- The goal is general chemical-class profiling of unannotated spectra (ClassyFire/ontology work), not accuracy measurement of structure ranking

## Inputs

- MS/MS spectra (HILIC-Q Exactive platform) for compounds identified at MSI confidence level 1 using authentic standards with precursor mass, retention time, and MS/MS matching
- List of known compound identities (authentic-standard assignments) for the same spectra
- Structure search space definition (biodatabase)
- Precursor mass accuracy tolerance specification (5 ppm)

## Outputs

- Per-compound record: top-ranked CSI:FingerID prediction vs. known authentic-standard identity, with rank position of the true structure among candidates
- Summary agreement rates across the compound set (top-ranked accuracy, e.g. 40%; top-3 accuracy, e.g. 52%)

## How to apply

Load the unambiguously assigned MSI level 1 MS/MS spectra (here, 103 compounds from the urinary HILIC-Q Exactive assay) and submit each to CSI:FingerID through the SIRIUS pipeline, restricting the structure search to the biodatabase with a 5 ppm precursor mass tolerance. Retrieve the ranked candidate structure list for each query from the CSI:FingerID web service. For each compound, compare the top-ranked prediction against the known authentic-standard identity, recording whether the predicted structure agrees and at which rank position it appears among candidates. Summarize agreement rates across the full compound set: the published reference values are 40% top-ranked accuracy and 52% top-3 accuracy on the 103-compound MSI level 1 set. Treat any structure assignment from Sirius/CSI:Finger ID as hypothesis-level (MSI level 3) and requiring validation with authentic chemicals, so this benchmark is the validation step rather than a substitute for standard-based identification.

## Related tools

- **CSI:FingerID** (In silico fragmentation tool that returns ranked candidate structures; its top-ranked predictions are the subject of the accuracy comparison against MSI level 1 compounds) — github.com/sirius-ms/sirius
- **Sirius/CSI:Finger ID** (SIRIUS pipeline (CLI/GUI/web service) that submits fragmentation trees and spectra to CSI:FingerID and retrieves candidate rankings) — github.com/sirius-ms/sirius
- **HILIC-Q Exactive HF mass spectrometry** (Instrument platform that acquired the MSI level 1 MS/MS spectra being benchmarked)

## Evaluation signals

- Computed top-ranked agreement rate on the 103-compound MSI level 1 set reproduces the published 40% accuracy
- Computed top-3 agreement rate reproduces the published 52% accuracy
- Every queried compound has both a CSI:FingerID candidate ranking and an authentic-standard (MSI level 1) identity for comparison
- Precursor mass tolerance is held at 5 ppm and the search space is fixed (biodatabase) so rankings are reproducible
- Agreement counts are computed only over unambiguously assigned spectra, excluding ambiguous or unannotated signals

## Limitations

- Top-ranked accuracy is modest (40%), so many structure assignments remain hypotheses that require validation by acquiring spectra from corresponding authentic chemicals
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS
- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals; only the 42% of urine signals that triggered MS/MS fragmentations are candidates for this benchmark

## Evidence
