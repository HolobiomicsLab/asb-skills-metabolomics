---
name: msms-precursor-fragment-association
description: Use when processing high-dimensional LC-IMS-MS/MS data where MS2 fragment
  ions must be attributed to MS1 precursors, especially when retention time alone
  is insufficient to separate coeluting species and drift time must be jointly considered.
when_to_use_negative:
- When the data are low-dimensional (only m/z and retention time, no drift time) —
  the paper shows that m/z versus RT followed by DT yields poor agreement and that
  retention time alone produces false assignments between 77 and 83 Da attributed
  to the m/z 212 precursor.
- When MS2 fragments have already been extracted and deconvolved — re-running association
  on an already deconvolved spectrum would double-count convolution artifacts rather
  than mitigate them.
- When the goal is feature detection or CCS calibration rather than fragment-to-precursor
  attribution — those are separate DEIMoS workflow steps.
edam_operation: http://edamontology.org/operation_3720
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Python API and command-line tool providing the deconvolution module whose
    cosine-similarity logic is reproduced for precursor–fragment association over
    LC-IMS-MS/MS data
  repo: https://github.com/pnnl/deimos
- name: Python
  role: Runtime environment in which the DEIMoS deconvolution module is invoked and
    the cosine similarity scoring is implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:f0d753a93948ef3c0588678e0ec65543efcc7d7149cb7b8beaa027b49aa344f7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pnnl/deimos
  repo_url: https://github.com/pnnl/deimos
  tools:
  - DEIMoS
  - LC-IMS-MS/MS
  - Python
  edam_operation: http://edamontology.org/operation_3720
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# MS/MS precursor–fragment association

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Associates MS2 fragment ions with their MS1 precursor ions by computing multidimensional cosine similarity scores across the non-m/z separation dimensions (drift time and retention time) of LC-IMS-MS/MS data, using DEIMoS deconvolution to mitigate convolution artifacts in tandem mass spectra.

## When to use

Use when processing high-dimensional LC-IMS-MS/MS data where MS2 fragment ions must be attributed to MS1 precursors, especially when retention time alone is insufficient to separate coeluting species and drift time must be jointly considered.

## When NOT to use

- When the data are low-dimensional (only m/z and retention time, no drift time) — the paper shows that m/z versus RT followed by DT yields poor agreement and that retention time alone produces false assignments between 77 and 83 Da attributed to the m/z 212 precursor.
- When MS2 fragments have already been extracted and deconvolved — re-running association on an already deconvolved spectrum would double-count convolution artifacts rather than mitigate them.
- When the goal is feature detection or CCS calibration rather than fragment-to-precursor attribution — those are separate DEIMoS workflow steps.

## Inputs

- MS1 precursor peak table from DEIMoS feature detection (HDF5, loaded via deimos.load)
- MS2 fragment peak table from DEIMoS feature detection (HDF5, loaded via deimos.load)
- LC-IMS-MS/MS data with drift time and retention time separation dimensions

## Outputs

- Table of retained MS1:MS2 pairings with multidimensional cosine similarity scores
- Deconvolved MS2 fragment assignments per MS1 precursor

## How to apply

Load the MS1 precursor and MS2 fragment peak tables already processed by DEIMoS feature detection and thresholding (e.g. from stored HDF5 data with deimos.load). For each candidate MS1:MS2 pairing, extract the separation profiles in the non-m/z dimensions (drift time and retention time) from the LC-IMS-MS/MS data and compute the cosine similarity between each precursor profile and each fragment profile across the joint drift time–retention time space, mirroring the deconvolution module logic (deimos.deconvolution). Retain only pairings whose similarity score exceeds the user-defined tolerance, thereby assigning fragments to precursors and mitigating convolution artifacts. The rationale, grounded in the paper, is that relying on retention time alone yields false assignments — masses between 77 and 83 Da would be wrongly attributed to the m/z 212 precursor — whereas joint drift time plus retention time similarity resolves the ambiguity. Output the retained pairings with their similarity scores as a single table.

## Related tools

- **DEIMoS** (Python API and command-line tool providing the deconvolution module whose cosine-similarity logic is reproduced for precursor–fragment association over LC-IMS-MS/MS data) — https://github.com/pnnl/deimos
- **Python** (Runtime environment in which the DEIMoS deconvolution module is invoked and the cosine similarity scoring is implemented)

## Examples

```
ms1 = deimos.load("example_data_peaks.h5", key="ms1"); ms2 = deimos.load("example_data_peaks.h5", key="ms2"); scores = deimos.deconvolution.cosine_similarity(ms1, ms2, tol=[0.1, 0.7, 0.15])
```

## Evaluation signals

- The output table exists, parses correctly, and contains a similarity score for every retained precursor–fragment pairing.
- Every retained MS1:MS2 pairing has a similarity score exceeding the specified cosine similarity tolerance.
- Fragment ions in the 77–83 Da mass range are NOT attributed to the m/z 212 precursor despite coelution in retention time — verifying that drift time was jointly used.
- The retained pairings are consistent with those produced by DEIMoS's own deconvolution module on the same thresholded peak tables.
- No convolution artifacts remain in the resulting deconvolved MS2 fragment assignments.

## Limitations

- Retention time alone is unreliable: masses between 77 and 83 Da would be assigned to the precursor with m/z 212 using retention time, whereas they are sufficiently separated in drift time.
- Low-dimensional processing (m/z versus RT followed by DT) results in only ~13% intersection with 3D processing, so dimensional reduction undermines both feature detection confidence and deconvolution accuracy.
- MS-DIAL, the most comparable technique, agrees poorly with DEIMoS when processing m/z versus RT followed by DT.

## Evidence

- [results] cosine similarity for precursor–fragment association: [evidence span withheld — anchor status: not_found]
