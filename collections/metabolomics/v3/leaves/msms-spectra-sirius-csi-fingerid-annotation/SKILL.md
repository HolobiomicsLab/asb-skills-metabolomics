---
name: msms-spectra-sirius-csi-fingerid-annotation
description: Use when untargeted LC-MS/MS spectra (e.g. HILIC- or CSH-Q Exactive HF
  data) cannot be matched to spectral libraries at MSI level 1/2 and in silico structure
  candidates must be ranked for MSI level 3 annotation.
when_to_use_negative:
- The compound is already identified at MSI level 1 with authentic standards (precursor
  mass, retention time, and MS/MS matching) or MSI level 2 by precursor accurate mass
  and MS/MS library matching — in silico annotation is only for the remaining spectra
- Spectra lack high-resolution isotope patterns, since SIRIUS formula ranking depends
  on isotope patterns from high-resolution mass spectra
- A validated library hit with false-discovery-rate assessment is available, given
  the field 'lacks generally accepted and validated automated calculations of compound
  identification confidence levels with false-discovery rate assessments'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: SIRIUS
  role: Java-based framework that computes fragmentation trees and isotope-pattern-based
    formulas, then submits spectra to the CSI:FingerID web service via GUI or command
    line
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: In silico fragmentation tool that predicts a molecular fingerprint from the
    MS/MS spectrum and returns top-ranked candidate structures from its structure
    database
  repo: https://github.com/sirius-ms/sirius
- name: sirius-client-openAPI (PySirius/RSirius)
  role: OpenAPI SDKs for programmatically driving SIRIUS/CSI:FingerID runs and retrieving
    results from the REST service
  repo: https://github.com/sirius-ms/sirius-client-openAPI
- name: HILIC-Q Exactive HF mass spectrometry
  role: Instrument platform that acquired the untargeted urinary MS/MS spectra benchmarked
    against CSI:FingerID top-hit accuracy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:7a23d4fb981841b8036da2ac16cb52dfcf9268994c3252c50c4f2cfe1638180b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - HILIC-Q Exactive HF mass spectrometry
  - SIRIUS
  - Sirius/CSI:Finger ID
  - sirius-client-openAPI (PySirius/RSirius)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msms-spectra-sirius-csi-fingerid-annotation

## Summary

Annotate MS/MS spectra of metabolites by submitting precursor masses and fragmentation patterns to SIRIUS/CSI:FingerID, retrieving top-ranked candidate structures from the structure database, and validating predictions against authentic-standard (MSI level 1) assignments. In the source paper this yielded 40% top-hit accuracy on 103 unambiguously assigned urinary HILIC-Q Exactive spectra.

## When to use

Use when untargeted LC-MS/MS spectra (e.g. HILIC- or CSH-Q Exactive HF data) cannot be matched to spectral libraries at MSI level 1/2 and in silico structure candidates must be ranked for MSI level 3 annotation.

## When NOT to use

- The compound is already identified at MSI level 1 with authentic standards (precursor mass, retention time, and MS/MS matching) or MSI level 2 by precursor accurate mass and MS/MS library matching — in silico annotation is only for the remaining spectra
- Spectra lack high-resolution isotope patterns, since SIRIUS formula ranking depends on isotope patterns from high-resolution mass spectra
- A validated library hit with false-discovery-rate assessment is available, given the field 'lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments'

## Inputs

- MS/MS peak lists or LC-MS/MS data files from HILIC-Q Exactive HF (or CSH-Q Exactive HF) runs, with precursor accurate mass and isotope pattern
- Structure search space / structure database (e.g. the biodatabase)
- Optional: authentic-standard (MSI level 1) compound identity table for validation

## Outputs

- Ranked list of candidate structures per MS/MS spectrum with the top-ranked CSI:FingerID prediction
- Predicted molecular formula and fragmentation tree from SIRIUS
- Agreement statistics (top-hit and top-3 accuracy) versus authentic-standard assignments

## How to apply

Run SIRIUS (java-based framework, GUI or CLI) on each MS/MS spectrum: first deduce the molecular formula from the high-resolution isotope pattern and fragmentation tree, then query CSI:FingerID via its web service using a structure search space such as the biodatabase with a precursor mass tolerance of 5 ppm (matching the Q Exactive HF accurate-mass regime). Retrieve the top-ranked candidate structures, recording each candidate's rank position. Validate by comparing predictions against the authentic-standard assignments: in the benchmark, 41 of 103 unambiguously assigned MSI level 1 spectra (40%) were correctly annotated as the top hit, and 52% within the top 3. Treat CSI:FingerID outputs as MSI level 3 annotations that require confirmation with authentic chemicals; for Python/R automation use the PySirius/RSirius OpenAPI SDKs.

## Related tools

- **SIRIUS** (Java-based framework that computes fragmentation trees and isotope-pattern-based formulas, then submits spectra to the CSI:FingerID web service via GUI or command line) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (In silico fragmentation tool that predicts a molecular fingerprint from the MS/MS spectrum and returns top-ranked candidate structures from its structure database) — https://github.com/sirius-ms/sirius
- **sirius-client-openAPI (PySirius/RSirius)** (OpenAPI SDKs for programmatically driving SIRIUS/CSI:FingerID runs and retrieving results from the REST service) — https://github.com/sirius-ms/sirius-client-openAPI
- **HILIC-Q Exactive HF mass spectrometry** (Instrument platform that acquired the untargeted urinary MS/MS spectra benchmarked against CSI:FingerID top-hit accuracy)

## Examples

```
sirius --input urine_msms.mgf --csi-fingerid --db biodatabase --ppm-max 5
```

## Evaluation signals

- Reproduce the published benchmark: 41/103 (40%) top-ranked agreement with MSI level 1 authentic-standard assignments when querying the biodatabase at 5 ppm mass accuracy
- Reproduce top-3 agreement rate of 52% for the same MSI level 1 compound set
- Each CSI:FingerID result must carry the candidate's rank position so top-hit vs top-3 rates can be computed
- Precursor mass tolerance must not exceed 5 ppm, consistent with Q Exactive HF accurate-mass filtering
- Results must be reported with MSI confidence levels, and level 3 assignments flagged for validation with authentic chemicals

## Limitations

- Only 40% of MSI level 1 spectra were correctly annotated as the top hit (52% in top 3), so most in silico annotations are not definitive
- Structure assignments from Sirius/CSI:Finger ID require validation with authentic chemicals before elevation to higher MSI confidence levels
- SIRIUS web services (CSI:FingerID, CANOPUS, MSNovelist) are restricted to academic research and education use; commercial use requires a Bright Giant license
- The field lacks validated automated confidence-level calculations with false-discovery-rate assessment, so reported accuracies depend on the specific benchmark set

## Evidence
