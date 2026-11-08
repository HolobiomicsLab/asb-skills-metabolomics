---
name: spectral-deconvolution-cosine-similarity
description: Use when processing MS/MS spectra from high-dimensional (e.g. LC-IMS-MS/MS)
  acquisitions where fragments must be attributed to precursor ions and retention
  time alone is insufficient to resolve overlapping features.
when_to_use_negative:
- When the data are 1D or 2D (m/z and retention time only, no ion mobility/drift time
  dimension) — the method relies on a second separation dimension to break precursor-fragment
  ambiguity.
- When input spectra are already deconvolved fragment-to-precursor assignments; re-running
  cosine scoring on such output would be redundant.
- When fragments cannot be paired to any candidate precursor because no joint drift
  time-retention time profile overlap exists at all.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DEIMoS
  role: Python API and command-line tool whose deconvolution module (deimos.deconvolution)
    performs the MS/MS spectral deconvolution; feature detection and thresholding
    steps (deimos.load, deimos.threshold) prepare the precursor and fragment peak
    tables fed into the cosine similarity scoring.
  repo: https://github.com/pnnl/deimos
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.1c05017
    title: 'DEIMoS: An Open-Source Tool for Processing High-Dimensional Mass Spectrometry
      Data'
  source_content_hash: sha256:c6e1c6b4e6c95746f3e3e097121037c7c48b437283a5747781c4b29d9d030e5a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-deconvolution-cosine-similarity

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign MS2 fragment ions to MS1 precursor ions in LC-IMS-MS/MS data by scoring precursor-fragment separation profiles (e.g. drift time and retention time) with cosine similarity in a joint non-m/z space, using DEIMoS's deconvolution module to mitigate convolution artifacts in tandem mass spectra.

## When to use

Use when processing MS/MS spectra from high-dimensional (e.g. LC-IMS-MS/MS) acquisitions where fragments must be attributed to precursor ions and retention time alone is insufficient to resolve overlapping features.

## When NOT to use

- When the data are 1D or 2D (m/z and retention time only, no ion mobility/drift time dimension) — the method relies on a second separation dimension to break precursor-fragment ambiguity.
- When input spectra are already deconvolved fragment-to-precursor assignments; re-running cosine scoring on such output would be redundant.
- When fragments cannot be paired to any candidate precursor because no joint drift time-retention time profile overlap exists at all.

## Inputs

- MS1 precursor peak table and MS2 fragment peak table loaded from DEIMoS-processed HDF5 data (e.g. example_data.h5, example_data_peaks.h5)
- Joint drift time and retention time profiles for each candidate MS1:MS2 pairing from LC-IMS-MS/MS data

## Outputs

- A single output table of per-MS1:MS2 pairing cosine similarity scores for pairs exceeding the user-defined similarity tolerance
- Deconvolved MS2 fragment assignments linking fragments to their precursor ions

## How to apply

Load the MS1 precursor and MS2 fragment peak tables (already processed by DEIMoS feature detection and thresholding, e.g. threshold=100) from the stored HDF5 data with deimos.load. For each candidate MS1:MS2 pairing, extract the separation profiles in the non-m/z dimensions (drift time and retention time) from the LC-IMS-MS/MS data and compute the cosine similarity between precursor and fragment profiles across the joint drift time-retention time space, mirroring the deimos.deconvolution module logic. Retain only pairings whose similarity score exceeds a user-defined tolerance; the article shows retention time alone would falsely assign 77-83 Da masses to the m/z 212 precursor, whereas drift time separation (e.g. dt_tol=0.04 in the isotopologue search tolerances) prevents such false attribution. Write the retained per-MS1:MS2 pairing similarity scores and the resulting deconvolved MS2 fragment assignments to a single output table.

## Related tools

- **DEIMoS** (Python API and command-line tool whose deconvolution module (deimos.deconvolution) performs the MS/MS spectral deconvolution; feature detection and thresholding steps (deimos.load, deimos.threshold) prepare the precursor and fragment peak tables fed into the cosine similarity scoring.) — https://github.com/pnnl/deimos

## Examples

```
ms1 = deimos.load("example_data.h5", key="ms1"); ms1 = deimos.threshold(ms1, threshold=100); ms2 = deimos.threshold(ms2, threshold=100)
```

## Evaluation signals

- The output score table exists, is parseable, and contains similarity scores for precursor-fragment pairings.
- Every retained pairing's similarity score exceeds the specified similarity tolerance.
- Fragment ions that share retention time but differ in drift time from the precursor (e.g. the 77-83 Da masses vs. the m/z 212 precursor) are NOT assigned to that precursor — the false assignment seen when using retention time alone is avoided.
- The number of retained MS1:MS2 pairings is strictly less than or equal to the number of candidate pairings considered, reflecting the filtering by similarity tolerance.

## Limitations

- Retention time alone is unreliable: the article explicitly shows that masses between 77 and 83 Da would be wrongly assigned to the m/z 212 precursor using retention time, so multiple non-m/z separation dimensions are required.
- The method assumes the MS1 and MS2 peak tables have already been processed by DEIMoS feature detection and thresholding; raw instrument data must go through those upstream steps first.
- The similarity tolerance is user-defined; an inappropriate threshold can either drop valid fragments or retain spurious pairings.

## Evidence

- [results] Retention time alone yields false assignments: the masses between 77 and 83 Da would be assigned to the precursor with m/z 212 using retention time, whereas they are sufficiently separated in drift time: [evidence span withheld — anchor status: not_found]
